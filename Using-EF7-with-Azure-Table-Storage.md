EF 7 includes support for Azure Table Storage (ATS). 

**This page does not include full instructions for setting up an EF7 project, read [Using EF7 in Traditional .NET Applications](https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Traditional-.NET-Applications/) for detailed info.**

## You need NuGet 2.8.3 or later
The EF7 NuGet packages use some new metadata that is only supported in NuGet 2.8.3 or higher. 

Note that NuGet version numbers can be confusing, while the first compatible release was branded 2.8.3 the product version of the extension is 2.8.50926.xxx.

* **Visual Studio 2015** - No updates needed, a compatible version of NuGet is included.
* **Visual Studio 2013** - You can [download a compatible version from Visual Studio Gallery](https://visualstudiogallery.msdn.microsoft.com/4ec1526c-4a8c-4a84-b702-b21a8f5293ca). A compatible version is also included in VS 2013 Update 4.
* **Visual Studio 2010 and 2012** - You can [download a compatible version from Visual Studio Gallery](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c).

**Make sure you restart Visual Studio after installing the update.**

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
            .ForAzureTableStorage()
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