# Configuring a DbContext

The purpose of this page is to identify the patterns that we provide for configuring a DbContext - primarily the datastore that it is connecting to.

This page identifies the primary patterns that we want to encourage developers to use - and thus will optimize the code base for. There are, of course, other patterns that are possible with the building blocks we provide.

There are two main buckets of scenarios
* **[Non Dependency Injection Scenarios](#non-dependency-injection-scenarios)** - These are mainly non-ASP.NET 5 applications, such as WPF, WinForms, Phone, Store, etc. Of course, you can also use a DI container in these applications, in which case the 'Dependency Injection Scenarios' apply.
* **[Dependency Injection Scenarios](#dependency-injection-scenarios)** - These are primarily about ASP.NET 5, where apps revolve around services being configured in a DI container in Startup.cs. 

# Non Dependency Injection Scenarios

* [Config inline in code](#config-inline-in-code)
* [Config from external code](#config-from-external-code)
* [Config inline in code but overridden from external code](#config-inline-in-code-but-overridden-from-external-code)
* [Connection from config rather than code](#connection-from-config-rather-than-code)

### Config inline in code

**Context code**

```
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnConfiguring(DbContextOptions options)
    {        options.UseSqlServer(@"Server=.\SQLEXPRESS;Database=Blogging;integrated security=True;");
    }
}
```

**Application code**

```
var context = new BloggingContext();
```

**Test code**

This approach doesn't really lend itself to testing (unless you target the full database). For testing, we would expect folks to "grow up" to one of the following two patterns.

## Config from external code

**Context Code**

```
public class BloggingContext : DbContext
{
    public CycleSalesContext(DbContextOptions options)
        : base(options)
    { }

    public DbSet<Blog> Blogs { get; set; }
}
```

**Application code**

```
var options = new DbContextOptions().UseSqlServer(@"Server=.\SQLEXPRESS;Database=Blogging;integrated security=True;");
var context = new BloggingContext(options);
```

**Test code**

```
var options = new DbContextOptions().UseInMemory();
var context = new BloggingContext(options);
```

## Config inline in code but overridden from external code

**Context Code**

```
public class BloggingContext : DbContext
{
    public BloggingContext ()
    { }

    public BloggingContext (DbContextOptions options)
        : base(options)
    { }

    public DbSet<Blog> Blogs { get; set; }

    protected override void OnConfiguring(DbContextOptions options)
    {        options.UseSqlServer(@"Server=.\SQLEXPRESS;Database=Blogging;integrated security=True;");
    }
}
```

**Application code**

```
var context = new BloggingContext();
```

**Test code**

**Open Issue:** Rather than locking the options could the developer just write defensive code in OnConfiguring to only set a data store if one isn't already set.

```
var options = new DbContextOptions().UseInMemory();
options.Lock(); 
var context = new BloggingContext(options);
```

## Connection from config rather than code

**Open Issue:** This scenario isn't really supported as yet. You need to use one of the approaches above but read the connection string from config.

```
Options.UseSqlServer(ConfigurationManager.ConnectionStrings["Blogging"].ConnectionString)
```


# Dependency Injection Scenarios
 
* [Config inline in code](#Config-inline-in-code)
* [Config in startup code, default ctor](#Config-in-startup-code-default-ctor)
* [Config in startup code, options ctor](#Config-in-startup-code-options-ctor)
* [Connection from config, provider specified in code](#Connection-from-config-provider-specified-in-code)
* [Connection and provider from config (longer term goal)](#Connection-and-provider-from-config-longer-term-goal)
* [ASP.NET vNext template](#ASP.NET-vNext-template)

## Config inline in code

**Startup code**

```
services.AddEntityFramework()
    .AddSqlServer()
    .AddDbContext<BloggingContext>();
```

**Context code**

```
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnConfiguring(DbContextOptions options)
    {        options.UseSqlServer(@"Server=.\SQLEXPRESS;Database=Blogging;integrated security=True;");
    }
}
```

**Application code**

```
public MyController(BloggingContext context)
```

## Config in startup code, default ctor

**Startup code**

```
services.AddEntityFramework()
    .AddSqlServer()
    .AddDbContext<BloggingContext>(options => 
        options.UseSqlServer(@"Server=.\SQLEXPRESS;Database=Blogging;integrated security=True;"));
```

**Context code**

```
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
}
```

**Application code**

```
public MyController(BloggingContext context)
```

## Config in startup code, options ctor

**Startup code**

```
services.AddEntityFramework()
    .AddSqlServer()
    .AddDbContext<BloggingContext>(options => 
        options.UseSqlServer(@"Server=.\SQLEXPRESS;Database=Blogging;integrated security=True;"));
```

**Context code**

**Note:** This should also work if there are additional constructor parameters after the 'options'. Those additional parameters should be resolved from the DI container.

```
public class BloggingContext : DbContext
{
    public BloggingContext (DbContextOptions options)
        : base(options)
    { }

    public DbSet<Blog> Blogs { get; set; }
}
```

**Application code**

```
public MyController(BloggingContext context)
```

**Test code**

```
var options = new DbContextOptions().UseInMemory();
var context = new BloggingContext(options);
```

## Connection from config, provider specified in code
This approach can be used in any of the above scenarios, rather than specifying the connection in code.

**Config.json**

```
"EntityFramework": {
    "BloggingContext": { 
        "ConnectionString" : "Server=.\SQLEXPRESS;Database=Blogging;integrated security=True;"
    }
}
```

The items inside the context element are just key/value pairs that are interpreted by the provider (i.e. there is no baked in concept of ConnectionString as not all providers require it). To begin with developers will still need to specify the provider in code via the options API, but in the future we would like to enable specifying the provider itself from config.

**Note:** We will also support loading the connection string from another config value:

```
"Data": {
    "DefaultConnection": { 
        "ConnectionString": "Server=(localdb)\\mssqllocaldb;Database=aspnetvnext-Blogging-259b5360-e4c0-4bc8-a417-bdc627b9d02b;Trusted_Connection=True;MultipleActiveResultSets=true"
    }
},
"EntityFramework": {
    "BloggingContext": { 
        "ConnectionStringKey" : "Data:DefaultConnection:ConnectionString"
    }
}
```

**Context or startup code**

```
options.UseSqlServer();
```

This method would decide what to do based on the key/value pairs from config. In the case of SQL Server it would just look for ConnectionString or ConnectionStringKey.

## Connection and provider from config (longer term goal)

**Config.json**

```
"EntityFramework": {
    "BloggingContext": { 
        "DataStore" : "EntityFramework.SqlServer",
        "ConnectionString" : "Server=.\SQLEXPRESS;Database=Blogging;integrated security=True;"
    }
}
```

Format is just an approximation, we still need to work out how you would specify the provider to use.

**Startup code**

```
services.AddEntityFramework()
    .AddSqlServer()
    .AddDbContext<BloggingContext>();
```

**Context code**

```
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
}
```

**Application code**

```
public MyController(BloggingContext context)
```


# ASP.NET vNext template

What we scaffold by default in project templates is important because it is the most likely to be copied and will be viewed as best practice. This summarizes the code we will scaffold for the ```ApplicationDbContext``` in new ASP.NET vNext projects.

**Startup code**

```
services.AddEntityFramework(config)
    .AddSqlServer()
    .AddDbContext<ApplicationDbContext>();
```

**Context code**

```
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnConfiguring(DbContextOptions options)
    {        options.UseSqlServer();
    }
}
```

**Config.json**

**Note:** The use of 'DefaultConnection' is important because there is logic in deployment that special cases this connection string to help with SQL Azure provisioning. 

```
"Data": {
    "DefaultConnection": { 
        "ConnectionString": "Server=(localdb)\\mssqllocaldb;Database=aspnetvnext-Blogging-259b5360-e4c0-4bc8-a417-bdc627b9d02b;Trusted_Connection=True;MultipleActiveResultSets=true"
    }
},
"EntityFramework": {
    "ApplicationDbContext": { 
        "ConnectionStringKey" : "Data:DefaultConnection:ConnectionString"
    }
}
```