EF 7 includes support for Azure Table Storage (ATS). This provide is a layer on top of the [.NET Client for Azure Storage](http://go.microsoft.com/fwlink/?linkid=390731&clcid=0x409). Below are some instructions for configuring EF to use ATS.

ATS is not a traditional relational data store, and careless use of EF may result in inefficient queries. Whenever possible, always include a partition key and row key in queries.

# Install the package
The ATS provider is bundled in the **EntityFramework.AzureTableStorage** package. The latest nightly build can be installed from using our [MyGet feed](https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Traditional-.NET-Applications#add-nightly-nuget-feed).

# Configuring the Model
A model must be configured in two places to ATS.

First, provide a connection string in the **OnConfiguring** method. 

Second, configure the partition and row key for your entity in the **OnModelCreating** method.

Sample:
```c#
public class AtsContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }

    protected override void OnConfiguring(DbContextOptions options)
    {
        options.UseAzureTableStorage("accountName", "accountKey");
          // OR
        options.UseAzureTableStorage("AccountName=accountName;AccountKey=accountKey;");
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.Entity<Customer>()
            .PartitionAndRowKey(c => c.PartitionKey, c => c.RowKey);
    }
        
}

public class Customer
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
 ...
}
```

For more detailed instructions on creating your model, see ["Create Your Model"](https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Traditional-.NET-Applications#create-your-model)

# Configuring the connection
ATS provides some configuration options.

### Batching
By default, set to false. When true, all additions, changes, and deletions will be send over HTTP as a batch request instead of individually. These batches follow the same behavior as batch requests made directly against [the REST API](http://msdn.microsoft.com/en-us/library/dd179355.aspx), including limitations in batch size and roll-back behavior.

```c#
using (var db = new AtsContext())
{
...
     db.Database.Connection.UseBatching(true);
...
}
```

### Request options
The same request options available from the .NET Azure Storage Client can be used here. For more details on those options, see [the documentation for the .NET client](http://go.microsoft.com/fwlink/?linkid=390731&clcid=0x409).

```c#

using (var db = new AtsContext())
{
...
     var requestOptions = new TableRequestOptions
     {
         ServerTimeout = TimeSpan.FromSeconds(42),
         LocationMode = LocationMode.PrimaryThenSecondary
     };
     db.Database.Connection.UseRequestOptions(requestOptions);

...

      db.Database.Connection.ResetRequestOptions();

...

}
```