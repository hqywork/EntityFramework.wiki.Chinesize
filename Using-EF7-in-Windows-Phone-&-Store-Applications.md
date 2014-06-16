# Using EF7 in Windows Phone & Store Applications

## Phone / Universal Not Yet Supported  
We’re still working thru some issues in order to make EF7 work on Windows Phone and Universal applications. This is due to some of our dependencies not yet supporting the wpa81 target platform.  

## Add Nightly NuGet Feed
You need to configure NuGet to use the feed that contains nightly builds.

1. In Visual Studio select **Tools –> NuGet Package Manager –> Package Manager Settings**
* Select **Package Sources** from the left pane 
* Select the **+** button to add a new source
* Enter **Nightly Builds** as the Name and **https://www.myget.org/F/aspnetvnext/api/v2/** as the Source 
* Click **Add** and then **OK**

## Install Entity Framework (and Providers)
To get EF7 in your project you need to install the **Microsoft.Data.Entity** NuGet package from the nightly feed. To do this, you can run the following command in Package Manager Console (Tools -> NuGet Package Manager -> Package Manager Console).
```
PM> Install-Package Microsoft.Data.Entity –Pre
```

You will also need to install a provider to be able to use EF to access a data store. Currently, the following provider packages are available for Windows Store applications:
* Microsoft.Data.Entity.SQLite
* Microsoft.Data.Entity.InMemory

## Additional Steps for SQLite

To use SQLite you will need to install the [SQLite for Windows Runtime extension](http://visualstudiogallery.msdn.microsoft.com/1d04f82f-2fe9-4727-a2f9-a2db127ddc9a).

Once you have installed the extension (and restarted Visual Studio):

1. **Project -> Add Reference...**
* Select **Extensions** from the left pane
* Check the **SQLite for Windows Runtime** box
* Click **OK**

## Create Your Model
Define a context and classes that make up your model. Note the new **OnConfiguring** method that is used to specify the data store provider to use (and, optionally, other configuration too). The following code uses the **Microsoft.Data.Entity.SQLite** provider.

**Note: The SQLite provider is currently very limited and does not support foreign keys or database generated values. Hence the very simple model used in this sample.**

```
using Microsoft.Data.Entity;  
using Microsoft.Data.Entity.Metadata;

namespace Sample
{
    public class BloggingContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }

        protected override void OnConfiguring(DbContextOptions builder)
        {
            var dir = Windows.Storage.ApplicationData.Current.LocalFolder.Path;
            var connection = "Filename=" + System.IO.Path.Combine(dir, "Blogging.db");

            builder.UseSQLite(connection);
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.Entity<Blog>()
                .Key(b => b.Url);
        }
    }

    public class Blog
    {
        public string Url { get; set; }
        public string Name { get; set; }
    }
}
```

## Use Your Model
You can now use your model to perform data access. Note that there is currently no Migrations support, so you will need to use the **DbContext.Database** API to explicitly create the database if it doesn’t exist.

```  
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