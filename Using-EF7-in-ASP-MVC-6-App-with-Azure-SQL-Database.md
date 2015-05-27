# Using EF7 in ASP MVC 6 Applications with Azure SQL Database
## Beta 4 is recommended
These instructions are for using EF7 Beta 4, which is the currently recommended release for trying out EF7 in Full .NET applications. 

You can find nightly builds of the EF7 code base hosted on **https://www.myget.org/F/aspnetvnext/api/v2/** but the code base is rapidly changing and we do not maintain up-to-date documentation for getting started.
## App
With this article, you will understand how to create ASP.NET 5 Web.API applications and combine them with the computing power Azure, the application will be based on the ASP.NET MVC and use EntityFramework 7 to store data. With a little effort, you can modify the application to work with a conventional ASP.NET MVC 6.

## project.json
First add the missing packages in your project file, it will be EF7 and MVC6 packages:
```json
"dependencies": {
        "Microsoft.AspNet.Server.IIS": "1.0.0-beta4",
        "Microsoft.AspNet.Server.WebListener": "1.0.0-beta4",
        "Microsoft.AspNet.Mvc": "6.0.0-beta4",
        "EntityFramework.SqlServer": "7.0.0-beta4",
        "EntityFramework.Commands": "7.0.0-beta4"
    }
```
Then be sure to add the following command, if it is not added to it, you may have difficulty using the EF7 migration:
```json
"commands": {
        "web": "~",
        "ef": "EntityFramework.Commands"
    },
```
## Configure Startup.cs
The application will use MVC if you add the service and run at the start of the MVC:
```C#
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseMvc();
        }
    }
```
## Create Your Model
Identify two unpretentious class: Post and Blog (which will contain a collection of Posts). These classes are designed for once something that would continue to use them with EF7
#### Post
```C#
public class Post
    {
        public int PostId { get; set; }
        public string Title { get; set; }
        public string Comment { get; set; }

        public int BlogId { get; set; }
    }
```
#### Blog
```C#
public class Blog
    {
        public int BlogId { get; set; }
        public string Name { get; set; }

        public List<Post> Posts { get; set; }
    }
```
## Create Your Controller
Create a controller containing a static List<Blog> with information about blogs and the two methods, one to create a blog, the other for get the collection of all blogs. Note the routing for that would not make a mistake during the test requests.
```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNet.Mvc;
using ASPNET5WebApi.Models;

namespace ASPNET5WebApi.Controllers
{
    [Route("api/[controller]/[action]")]
    public class BloggingController
    {
        private static readonly List<Blog> _Blogs = new List<Blog>()
        {
            new Blog() {
                Id =1,
                Name ="ExampleBlog",
                Posts=new List<Post>()
                {
                    new Post() { Id=Guid.NewGuid().ToString(), Comment="Example" },
                    new Post() { Id=Guid.NewGuid().ToString(), Comment="Blog" }
                }
            }
        };

        [HttpGet]
        public Object addBlog()
        {           
            var MaxBlogId = _Blogs.Max(x => x.Id);

            Blog NewBlog = new Blog() { Id = ++MaxBlogId };
            NewBlog.Name = "Testblog #" + MaxBlogId.ToString();

            Post FirstPost = new Post() { Id = Guid.NewGuid().ToString(), Comment = "Test" };
            Post SecondPost = new Post() { Id = Guid.NewGuid().ToString(), Comment = "blog" };

            NewBlog.Posts = new List<Post>() { FirstPost, SecondPost };

            _Blogs.Add(NewBlog);

            return _Blogs;
        }

        [HttpGet]
        public Object getBlogs()
        {
            return _Blogs;
        }
    }
}
```
## Deploy to Azure
* Right click on project
* Deploy
* Choose Azure web hosting
* Create new site
* Auth if needed
* Parameters
* Uncheck "Using PowerShell"
* Publish

Before you go further, try browse `[sitename].azurewebsites.net/api/getblogs` for run `[sitename].azurewebsites.net/api/getblogs` and `[sitename].azurewebsites.net/api/getblogs` to test the api.
## Connect EntityFramework to Azure
Using this method, you can connect to the EF to your Web.API application
### Add BloggingContext
If you use the AzureSQL V11 is required to add `.ForSqlServer().UseIdentity()` to the `OnModelCreating` method. If you use AzureSQL V12 can try without it. In order to find out the connection string to go to Azure control panel, then select your chosen database and click on the "get a connection string", a window opens in which you want to copy the connection string ADO.NET. In this example, it is not considered a cyclic model (specially).
```C#
   public class BloggingContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(@"AzureSQLConnectionString");
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.ForSqlServer().UseIdentity();
        }
    }
```
### Using migrations
After installation EF in Visual Studio 2015RC possible you can have problems with the migration, if it's not true skip this part and use the `PM>Add Migration` to migration. If you have an error similar to (https://github.com/aspnet/EntityFramework/issues/1196) then follow the instructions below.
* Install dnx http://docs.asp.net/en/latest/getting-started/installing-on-windows.html#install-the-net-execution-environment-dnx
* Use command prompt: `cd solution/project/src/project`

For the next step, we need to use project command, make sure to add it to the project.json
* Add your first migration: `dnx . ef migration add InitMigration`
For the next step, we need to connect to Azure, if you are unsure of your connection string, you can once again open the deployment on Azure, and find the settings button to verify the connection. If you have successfully connected to your database, continue.
* Apply your migration: `dnx . ef migration apply`

## Update Your Controller
Now we want to update the controller for use with the AzureSQL database:
```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNet.Mvc;
using ASPNET5WebApi.Models;

namespace ASPNET5WebApi.Controllers
{
    [Route("api/[controller]/[action]")]
    public class BloggingController
    {
        [HttpGet]
        public Object addBlog()
        {
            using (var db = new BloggingContext())
            {
                var MaxBlogId = db.Blogs.Max(x => x.BlogId);

                Blog NewBlog = new Blog();
                NewBlog.Name = "Testblog #" + MaxBlogId.ToString();

                Post FirstPost = new Post() { Comment = "Test" };
                Post SecondPost = new Post() { Comment = "blog" };

                

                NewBlog.Posts = new List<Post>() { FirstPost, SecondPost };

                db.Blogs.Add(NewBlog);
                db.Posts.Add(FirstPost);
                db.Posts.Add(SecondPost);
                
                db.SaveChanges();

                return true;
            }
        }


        [HttpGet]
        public Object getBlogs()
        {
            using (var db = new BloggingContext())
            {
                var AllBlogs = db.Blogs.ToList();
                foreach (var Blog in AllBlogs)
                    Blog.Posts = new List<Post>(from x in db.Posts where x.BlogId == Blog.BlogId select x);
                return AllBlogs;
            }
        }
    }
}
```
Now you can try `[sitename].azurewebsites.net/api/addblog` and then `[sitename].azurewebsites.net/api/getblogs`, if everything is correct and you used exactly the versions of packages that are described here, you successfully create a new item and get it through the method of `getBlogs()`.

## Sources
You can find the source code of the project here : https://github.com/anetegithub/ASP.NET-5-EF-7-AzureSQL-V11
This article used information from:
* https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Traditional-.NET-Applications
* http://www.codeproject.com/Tips/988763/Database-Migration-in-Entity-Framework
* https://github.com/aspnet/EntityFramework/issues/2278
