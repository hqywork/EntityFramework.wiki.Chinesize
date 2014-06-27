# Getting Started with EF7 Nightly Builds
If you want to get involved with EF7 you can try out our nightly builds. 

## Limitations

**As you try out EF7 please bear in mind that this is a very early stage in the development of the new EF codebase and there are many features that are partially implemented or not yet available.**

In particular, there are the following limitations:

* Data Annotations are not yet supported for configuring a model.
* The Fluent API only has very basic functionality implemented so far. For many modelling concerns (such as defining relationships between entities) you need to manipulate the underlying object model directly.
* The relational database providers process some parts of the query in the database and others in-memory. There are a number of operations currently performed in-memory that will be moved to database evaluation as we progress with EF7.
* There is currently no loading of related data (eager, lazy, or explicit loading). To load related data you need to issue a separate LINQ query.
* Migrations are not yet supported – although we do have a lot of the building blocks in place. Currently, you’ll need to use the DbContext.Database API to maintain your database.

## Using EF7 in Traditional .NET Applications

You can use the nightly builds of EF7 in your traditional .NET applications. This includes Console Applications, WPF, WinForms, ASP.NET, etc. see [Using EF7 in Traditional .NET Applications](https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Traditional-.NET-Applications) for instructions.

## Using EF7 in Windows Store/Phone Applications

For details on using EF7 in Windows Phone and Windows Store applications, see [Using EF7 in Windows Phone/Store Applications](https://github.com/aspnet/EntityFramework/wiki/Using-EF7-in-Windows-Phone-&-Store-Applications) for instructions.

## Using EF7 with ASP.NET vNext
EF7 can be used in applications built using ASP.NET vNext. You can find samples, documentation and getting started instructions for ASP.NET vNext at the [Home](https://github.com/aspnet/Home) repo. 

You can see a sample MVC application that uses EF7 for data access in the [MusicStore](https://github.com/aspnet/MusicStore) repo. There are also a couple of micro samples in the [Entropy](https://github.com/aspnet/Entropy) repo – Data.SqlServer and Data.InMemory.