# Using EF7 in Traditional .NET Applications

## Beta 4 is recommended
These instructions are for using EF7 Beta 4, which is the currently recommended release for trying out EF7 in Full .NET applications. 

You can find nightly builds of the EF7 code base hosted on **https://www.myget.org/F/aspnetvnext/api/v2/** but the code base is rapidly changing and we do not maintain up-to-date documentation for getting started.

## You need NuGet 2.8.5 or later
The EF7 NuGet packages use some new metadata that is only supported in NuGet 2.8.5 or higher. 

_Note that NuGet version numbers can be confusing, while the first compatible release was branded 2.8.5 the product version of the extension is 2.8.60318.xxx._

* **Visual Studio 2015** - No updates needed, a compatible version of NuGet is included.
* **Visual Studio 2013** - You can [download a compatible version from Visual Studio Gallery](https://visualstudiogallery.msdn.microsoft.com/4ec1526c-4a8c-4a84-b702-b21a8f5293ca).
* **Visual Studio 2010 and 2012** - You can [download a compatible version from Visual Studio Gallery](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c).

**Make sure you restart Visual Studio after installing the update.**

## Install Entity Framework (and Providers)
To get EF7 in your project you need to install the package for the database provider(s) you want to target. Currently, the following provider packages are available for full .NET applications:
* EntityFramework.SqlServer
* EntityFramework.InMemory

The sample on this page uses SQL Server, so you would run the following command in Package Manager Console. 

```
PM> Install-Package EntityFramework.SqlServer –Pre
```

## Create Your Model
Define a context and classes that make up your model. Note the new **OnConfiguring** method that is used to specify the data store provider to use (and, optionally, other configuration too).

```csharp
using System.Collections.Generic;
using Microsoft.Data.Entity;

namespace Sample
{
    public class BloggingContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            // Visual Studio 2015 | Use the LocalDb 12 instance created by Visual Studio
            optionsBuilder.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;");

            // Visual Studio 2013 | Use the LocalDb 11 instance created by Visual Studio
            //optionsBuilder.UseSqlServer(@"Server=(localdb)\v11.0;Database=Blogging;Trusted_Connection=True;");

            // Visual Studio 2012 | Use the SQL Express instance created by Visual Studio
            //optionsBuilder.UseSqlServer(@"Server=.\SQLEXPRESS;Database=Blogging;Trusted_Connection=True;");
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.Entity<Blog>()
                .Collection(b => b.Posts)
                .InverseReference(p => p.Blog)
                .ForeignKey(p => p.BlogId);
        }
    }

    public class Blog
    {
        public int BlogId { get; set; }
        public string Url { get; set; }

        public List<Post> Posts { get; set; }
    }

    public class Post
    {
        public int PostId { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public Blog Blog { get; set; }
    }
}
```

## Create Your Database
Now that you have a model, you can use migrations to create a database for you.

In Package Manager Console (**Tools –> NuGet Package Manager –> Package Manager Console**):

1. ```Install-Package EntityFramework.Commands -Pre``` to make the migrations commands available in your project.
* ```Add-Migration MyFirstMigration``` to scaffold a migration to create the initial set of tables for your model.
* ```Apply-Migration``` to apply the new migration to the database. Because your database doesn't exist yet, it will be created for you before the migration is applied.

If you make future changes to your model, you can use the ```Add-Migration``` command to scaffold a new migration to apply the corresponding changes to the database. Once you have checked the scaffolded code (and made any required changes), you can use the ```Apply-Migration``` command to apply the changes to the database.

## Use Your Model
You can now use your model to perform data access.

```csharp
using System;

namespace Sample
{
    class Program
    {
        static void Main(string[] args)
        {
            using (var db = new BloggingContext())
            {
                db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/adonet" });
                db.SaveChanges();

                foreach (var blog in db.Blogs)
                {
                    Console.WriteLine(blog.Url);
                }
            }
        }
    }
}
```