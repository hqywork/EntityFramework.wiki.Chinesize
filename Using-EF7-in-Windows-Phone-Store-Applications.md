# Using EF7 in Windows Phone Store Applications

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

You will also need to install a provider to be able to use EF to access a data store. Currently, the following provider packages are available for Windows Store applications:
* Microsoft.Data.Entity.SQLite
* Microsoft.Data.Entity.InMemory

## Create Your Model
Define a context and classes that make up your model. Note the new **OnConfiguring** method that is used to specify the data store provider to use (and, optionally, other configuration too). The following code uses the **Microsoft.Data.Entity.SQLite** provider.

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
            var dir = Windows.Storage.ApplicationData.Current.LocalFolder.Path;
            var connection = "Filename=" + System.IO.Path.Combine(dir, "Blogging.db");

            builder.UseSQLite(connection);
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

Note: When using SQLite, the **Database.Exists** method is not implemented as yet. You can safely call **Database.Create** though and it will be a no-op if the database already exists.

```
public App()
{
    this.InitializeComponent();
    this.Suspending += this.OnSuspending;

    using (var db = new BloggingContext())
    {
        db.Database.Create();
    }
}
```

```
private void Page_Loaded(object sender, Windows.UI.Xaml.RoutedEventArgs e)
{
    using (var db = new BloggingContext())
    {
        this.BlogsListView.ItemsSource = db.Blogs.ToList();
    }
}
```