# Using EF7 in Traditional .NET Applications

## You need NuGet 2.8.3 or later
The EF7 NuGet packages use some new metadata that is only supported in NuGet 2.8.3 or higher. 

Note that NuGet version numbers can be confusing, while the first compatible release was branded 2.8.3 the product version of the extension is 2.8.50926.xxx.

* **Visual Studio 2015** - No updates needed, a compatible version of NuGet is included.
* **Visual Studio 2013** - You can [download a compatible version from Visual Studio Gallery](https://visualstudiogallery.msdn.microsoft.com/4ec1526c-4a8c-4a84-b702-b21a8f5293ca). A compatible version is also included in VS 2013 Update 4.
* **Visual Studio 2010 and 2012** - You can [download a compatible version from Visual Studio Gallery](https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c).

**Make sure you restart Visual Studio after installing the update.**

## Add Nightly NuGet Feed
You need to configure NuGet to use the feed that contains nightly builds.

1. In Visual Studio select **Tools –> NuGet Package Manager –> Package Manager Settings**
* Select **Package Sources** from the left pane 
* Select the **+** button to add a new source
* Enter **Nightly Builds** as the Name and **https://www.myget.org/F/aspnetvnext/api/v2/** as the Source 
* Click **Add** and then **OK**

## Install Entity Framework (and Providers)
To get EF7 in your project you need to install the package for the database provider(s) you want to target. Currently, the following provider packages are available for full .NET applications:
* EntityFramework.SqlServer
* EntityFramework.SQLite
* EntityFramework.AzureTableStorage
* EntityFramework.InMemory

The sample on this page uses SQL Server, so you would run the following command in Package Manager Console. **Make sure the nightly feed you created in the previous step is selected before running this command**.

```
PM> Install-Package EntityFramework.SqlServer –Pre
```

**To use a relational database provider, your application will need to target .NET 4.5.1.**

## Create Your Model
Define a context and classes that make up your model. Note the new **OnConfiguring** method that is used to specify the data store provider to use (and, optionally, other configuration too). The following code uses the **EntityFramework.SqlServer** provider.

```csharp
using Microsoft.Data.Entity;
using Microsoft.Data.Entity.Metadata;
using System.Collections.Generic;
using System.Linq;

namespace Sample
{
    public class BloggingContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }

        protected override void OnConfiguring(DbContextOptions builder)
        {
            builder.UseSqlServer(@"Server=(localdb)\v11.0;Database=Blogging;Trusted_Connection=True;");
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.Entity<Blog>()
                .OneToMany(b => b.Posts, p => p.Blog)
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