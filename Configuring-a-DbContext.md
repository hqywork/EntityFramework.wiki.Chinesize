# Configuring a DbContext

The purpose of this page is to identify the patterns that we provide for configuring a DbContext - primarily the datastore that it is connecting to.

This page identifies the primary patterns that we want to encourage developers to use - and thus will optimize the code base for. There are, of course, other patterns that are possible with the building blocks we provide.

There are two main buckets of scenarios
* **Non Dependency Injection Scenarios** - These are mainly non-ASP.NET 5 applications, such as WPF, WinForms, Phone, Store, etc. Of course, you can also use a DI container in these applications, in which case the 'Dependency Injection Scenarios' apply.
* **Dependency Injection Scenarios** - These are primarily about ASP.NET 5, where apps revolve around services being configured in a DI container in Startup.cs. 

# Non Dependency Injection Scenarios

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