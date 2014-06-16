# Entity Framework
Entity Framework is Microsoft's recommended data access technology for new applications in .NET. 

Entity Framework 7 (EF7) provides a familiar developer experience to previous versions of EF, including LINQ, POCO, and Code First support. EF7 also enables access to data across relational and non-relational stores. EF7 is much more lightweight than previous versions and is built from the ground up to work great in the cloud (using ASP.NET vNext) on devices (i.e. in universal Windows apps) as well as in traditional .NET scenarios.


## Getting Started with EF7 Nightly Builds
If you want to get involved with EF7 you can try out our nightly builds. 

### Limitations

**As you try out EF7 please bear in mind that this is a very early stage in the development of the new EF codebase and there are many features that are partially implemented or not yet available.**

In particular, there are the following limitations:

* Data Annotations are not yet supported for configuring a model.
* The Fluent API only has very basic functionality implemented so far. For many modelling concerns (such as defining relationships between entities) you need to manipulate the underlying object model directly.
* The relational database providers currently issue a “SELECT *” query for every LINQ query and then do the filtering, shaping, etc. in-memory. We’ll soon be checking in the proper LINQ provider that delegates the appropriate parts of the query to the database.
* There is currently no loading of related data (eager, lazy, or explicit loading). To load related data you need to issue a separate LINQ query.
* Migrations are not yet supported – although we do have a lot of the building blocks in place. Currently, you’ll need to use the DbContext.Database API to maintain your database.
* We have demo’d a provider for Azure Table Storage, but this provider is not yet included in our public code base or the nightly builds. We will make it available shortly.

### Using EF7 in Traditional .NET Applications

You can use the nightly builds of EF7 in your traditional .NET applications. This includes Console Applications, WPF, WinForms, ASP.NET, etc. see [Using EF7 in Traditional .NET Applications](https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Traditional-.NET-Applications) for instructions.

### Using EF7 in Windows Store/Phone Applications

For details on using EF7 in Windows Phone and Windows Store applications, see [Using EF7 in Windows Phone/Store Applications](https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Windows-Phone-&-Store-Applications) for instructions.

### Using EF7 with ASP.NET vNext
EF7 can be used in applications built using ASP.NET vNext. You can find samples, documentation and getting started instructions for ASP.NET vNext at the [Home](https://github.com/aspnet/Home) repo. 

You can see a sample MVC application that uses EF7 for data access in the [MusicStore](https://github.com/aspnet/MusicStore) repo. There are also a couple of micro samples in the [Entropy](https://github.com/aspnet/Entropy) repo – Data.SqlServer and Data.InMemory.

## Additional Information
- [Design Meeting Notes](https://github.com/aspnet/EntityFramework/wiki/Entity-Framework-Design-Meeting-Notes)