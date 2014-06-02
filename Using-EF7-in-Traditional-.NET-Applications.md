# Using EF7 in Traditional .NET Applications

## Add Nightly NuGet Feed
You need to configure NuGet to use the feed that contains nightly builds.

1. In Visual Studio select **Tools –> NuGet Package Manager –> Package Manager Settings**
* Select **Package Sources** from the left pane 
* Select the **+** button to add a new source
* Enter **Nightly Builds** as the Name and **https://www.myget.org/F/aspnetvnext/api/v2/** as the Source 
* Click **Add** and then **OK**

## Install Entity Framework (and Providers)
To get EF7 in your project you need to install the **Microsoft.Data.Entity** NuGet package from the nightly feed. To do this, you can run the following command in Package Manager Console.
```
PM> Install-Package Microsoft.Data.Entity –Pre
```

You will also need to install a provider to be able to use EF to access a data store. Currently, the following provider packages are available for full .NET applications:
* Microsoft.Data.Entity.SqlServer
* Microsoft.Data.Entity.SQLite (currently not working)
* Microsoft.Data.Entity.InMemory

To use a relational database provider, you will need to target .NET 4.5.1.

## Create Your Model
Define a context and classes that make up your model. Note the new **OnConfiguring** method that is used to specify the data store provider to use (and, optionally, other configuration too). The following code uses the **Microsoft.Data.Entity.SqlServer** provider.

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
            // If you want to store the connection string in Web/App.config then use
            // ConfigurationManager to load it from config and pass the string to UseSqlServer.
            builder.UseSqlServer(@"Server=(localdb)\v11.0;Database=Blogging;Trusted_Connection=True;");
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            // This is an early API and will change to more usable
            builder.Entity<Post>()
                .ForeignKeys(b =>
                {
                    b.ForeignKey<Blog>(p => p.BlogId);
                });

            // Fluent API does not support configuring relationships yet. You
            // need to manipulate the underlying object model to define them.
            var blog = builder.Model.GetEntityType(typeof(Blog));
            var post = builder.Model.GetEntityType(typeof(Post));
            var fk = post.ForeignKeys.Single(f => f.Properties.Any(p => p.Name == "BlogId"));
            blog.AddNavigation(new Navigation(fk, "Posts"));
            post.AddNavigation(new Navigation(fk, "Blog"));
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

## Use Your Model
You can now use your model to perform data access. Note that there is currently no Migrations support, so you will need to use the **DbContext.Database** API to explicitly create the database if it doesn’t exist.

```
using System;

namespace Sample
{
    class Program
    {
        static void Main(string[] args)
        {
            using (var db= new BloggingContext())
            {
                // If the database doesn't exist, then create it.
                if(!db.Database.Exists())
                {
                    db.Database.Create();
                    db.Database.CreateTables();
                }

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