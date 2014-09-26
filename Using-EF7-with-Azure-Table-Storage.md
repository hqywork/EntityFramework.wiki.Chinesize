EF 7 includes support for Azure Table Storage (ATS). 

**This page does not include full instructions for setting up an EF7 project, read [Using EF7 in Traditional .NET Applications](https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Traditional-.NET-Applications/) for detailed info.**

## Install the package
The ATS provider is shipped as the **EntityFramework.AzureTableStorage** NuGet package. You can install this from the EF7 nightly build feed.

```
PM> Install-Package EntityFramework.AzureTableStorage –Pre
```

## Configuring the Model
A model that targets ATS must be configured in two places:

1. ATS connection string in the ```OnConfiguring``` method. 
* Partition and row key for each entity in the ```OnModelCreating``` method.

```c#
public class CustomerContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }

    protected override void OnConfiguring(DbContextOptions options)
    {
        options.UseAzureTableStorage("AccountName=<accountName>;AccountKey=<accountKey>;");
        // Alternatively you can also use the following overload:
        // options.UseAzureTableStorage("<accountName>", "<accountKey>");
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.Entity<Customer>()
            .PartitionAndRowKey(c => c.LastName, c => c.FirstName);
    }
}

public class Customer
{
    public string LastName { get; set; }
    public string FirstName { get; set; }
    public string Email { get; set; }
}
```

## Be Careful with LINQ
ATS is not a traditional relational data store, and careless use of LINQ may result in inefficient queries. Whenever possible, always include a partition key and row key in queries.

## Additional Configuration
The ATS provider allows some additional configuration options.

### Batching
By default, set to false. When true, all additions, changes, and deletions will be send over HTTP as a batch request instead of individually. These batches follow the same behavior as batch requests made directly against [the REST API](http://msdn.microsoft.com/en-us/library/dd179355.aspx), including limitations in batch size and roll-back behavior.

```c#
using (var db = new CustomerContext())
{
...
     db.Database.Connection.AsAtsConnection().UseBatching(true);
...
}
```

### Request options
The same request options available from the .NET Azure Storage Client can be used here. For more details on those options, see [the documentation for the .NET client](http://go.microsoft.com/fwlink/?linkid=390731&clcid=0x409).

```c#
using (var db = new CustomerContext())
{
...
     var requestOptions = new TableRequestOptions
     {
         ServerTimeout = TimeSpan.FromSeconds(42),
         LocationMode = LocationMode.PrimaryThenSecondary
     };
     db.Database.Connection.AsAtsConnection().UseRequestOptions(requestOptions);

...

      db.Database.Connection.AsAtsConnection().ResetRequestOptions();

...

}
```