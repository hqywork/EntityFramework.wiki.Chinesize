# Using EF7 in Windows Phone & Store Applications

## Supported App Types
EF7 works on **Windows Phone 8.1** and **Windows Store 8.1** applications, plus the new **Universal Apps** that target both these platforms.

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
* EntityFramework.SQLite
* EntityFramework.InMemory

The sample on this page uses SQLite, so you would run the following command in Package Manager Console. **Make sure the nightly feed you created in the previous step is selected before running this command**.

```
PM> Install-Package EntityFramework.SQLite –Pre
```

## Additional Steps for SQLite

To use SQLite you will need to install the SQLite runtime:
* [SQLite for Windows Phone 8.1](http://visualstudiogallery.msdn.microsoft.com/5d97faf6-39e3-4048-a0bc-adde2af75d1b)
* [SQLite for Windows Runtime (Windows 8.1) ](http://visualstudiogallery.msdn.microsoft.com/1d04f82f-2fe9-4727-a2f9-a2db127ddc9a)

Once you have installed the extension (and restarted Visual Studio):

1. **Project -> Add Reference...**
* Select **Extensions** from the left pane
* Check the **SQLite for Windows...** box
* Click **OK**

## Create Your Model
Define a context and classes that make up your model. Note the new **OnConfiguring** method that is used to specify the data store provider to use (and, optionally, other configuration too). 

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
#if WINDOWS_PHONE_APP
            var connection = "Filename=GameHistory.db";
#else 
            var dir = Windows.Storage.ApplicationData.Current.LocalFolder.Path; 
            var connection = "Filename=" + System.IO.Path.Combine(dir, "GameHistory.db"); 
#endif

            builder.UseSQLite(connection);
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.Entity<Blog>()
                .OneToMany(b => b.Posts, p => p.Blog)
                .ForeignKey(p => p.BlogId);

            // The EF7 SQLite provider currently doesn't support generated values
            // so setting the keys to be generated from developer code
            builder.Entity<Blog>()
                .Property(b => b.BlogId)
                .GenerateValueOnAdd(false);

            builder.Entity<Post>()
                .Property(b => b.BlogId)
                .GenerateValueOnAdd(false);
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

In desktop/server applications you would normally use the ```Apply-Migration``` to apply the new migration to your development database. However, because the database is going to be created on the devices local storage, you need to add some code to apply migrations on the device. You can do this during app startup:

```
using (var db = new BloggingContext())
{
   db.Database.AsMigrationsEnabled().ApplyMigrations();
}
```

Note you need to import the Microsoft.Data.Entity namespace to get the required extension methods; ```using Microsoft.Data.Entity;```.

Because your database doesn't exist yet, it will be created when the migration is applied.

If you make future changes to your model, you can use the ```Add-Migration``` command to scaffold a new migration to apply the corresponding changes to the database.

## Use Your Model
You can now use your model to perform data access. Note that there is currently no Migrations support for Phone/Store apps, so you will need to use the ```DbContext.Database``` API to explicitly create the database if it doesn’t exist.

```csharp
public App()  
{  
    this.InitializeComponent();  
    this.Suspending += this.OnSuspending;  
  
    using (var db = new BloggingContext())  
    {  
        // Migrations are not yet enabled in EF7, so use an
        // API to create the database if it doesn't exist
        db.Database.EnsureCreated();  
    }  
}  

private void Page_Loaded(object sender, Windows.UI.Xaml.RoutedEventArgs e)  
{  
    using (var db = new BloggingContext())  
    {  
        this.BlogsListView.ItemsSource = db.Blogs.ToList();  
    }  
}  
```