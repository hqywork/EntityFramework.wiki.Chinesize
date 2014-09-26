# Using EF7 in Traditional .NET Applications

## Install a Build of NuGet 2.8.3
The EF7 NuGet packages use some new metadata that isn't supported in currently released versions of NuGet. Until a version of NuGet 2.8.3 is released, here are the steps to install a nightly build that can work with the latest EF7 packages:

* In Visual Studio **Tools -> Extensions and Updates...**
* Select **Installed** from left pane
* Locate **NuGet Package Manager** and uninstall it
* Go to the latest build of [NuGet 2.8.3](https://nugetbuild.cloudapp.net/viewLog.html?buildTypeId=Client_VisualStudio_NuGetForVS2013&buildId=lastSuccessful&buildBranch=2.8.3&tab=artifacts#!19frxayyv0m10)
* Download **VisualStudioAddIn\NuGet.Tools-2013.vsix** (note it may try to save as a **.zip** file, if so you need to swap the extension to **.vsix**)
* Double click the VSIX and complete the install
* Restart VS

**Note this will break some functionality in Visual Studio, including creating new EDMX models using the EF6.x Designer.**

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

```
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
* ```Update-Database``` to apply the new migration to the database. Because your database doesn't exist yet, it will be created for you before the migration is applied.

If you make future changes to your model, you can use the ```Add-Migration``` command to scaffold a new migration to apply the corresponding changes to the database. Once you have checked the scaffolded code (and made any required changes), you can use the ```Update-Database``` command to apply the changes to the database.

## Use Your Model
You can now use your model to perform data access.

```
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