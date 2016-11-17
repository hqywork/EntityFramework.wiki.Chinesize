# Entity Framework Core (EF Core)

Below is the schedule and roadmap for EF Core. Please note that these dates and feature plans are all subject to change. As with any project of this size it is difficult to predict exactly when things will land. Even so, we think it's important to be as open and transparent as possible about our plans so that our users can have the right expectations and create their plans accordingly.

## Schedule

The schedule for EF Core is in-sync with the .NET Core and ASP.NET Core schedule [which you can find here](https://github.com/aspnet/Home/wiki/Roadmap). 

## Backlog

Because EF Core is a new code base, the presence of a feature in Entity Framework 6.x does not mean that the feature is implemented in EF Core. For more information, see [Compare EF Core & EF6.x](https://docs.microsoft.com/en-us/ef/efcore-and-ef6/index)

We have provided a list of features that we think are important but are not yet implemented. This is by no means an exhaustive list, but calls out some of the important features that are not yet implemented in EF Core.

### Critical O/RM features

The things we think we need before we say EF Core is the recommended version of EF. Until we implement these features EF Core will be a valid option for many applications, especially on platforms such as UWP and .NET Core where EF6.x does not work, but for many applications the lack of these features will make EF6.x a better option.

* Query
 * **Improved translation** will enable more queries to successfully execute, with more logic being evaluated in the database (rather than in-memory).
  * **GroupBy translation** will move translation of the LINQ GroupBy operator to the database, rather than in-memory.
 * **Lazy loading** enables navigation properties to be automatically populated from the database when they are accessed.
 * **Raw SQL queries for non-Model types** allows a raw SQL query to be used to populate types that are not part of the model (typically for denormalized view-model data).

* Database schema management 
 * **Visual Studio wizard for reverse engineer**, that allows you to visually configure connection, select tables, etc. when creating a model from an existing database.
 * **Update model from database** allows a model that was previously reverse engineered from the database to be refreshed with changes made to the schema.

* Modelling
 * **Complex/value types** are types that do not have a primary key and are used to represent a set of properties on an entity type.

* Change Tracking
* Relational specific
 * **Stored procedure mapping** allows EF to use stored procedures to persist changes to the database (`FromSql` already provides good support for using a stored procedure to query).
 * **View mapping** allows EF to map to database views.
 
### High priority features

There are many features on our backlog and this is by no means an exhaustive list. These features are high priority but we think EF Core would be a compelling release for the vast majority of applications without them

* Modelling
 * **More flexible property mapping**, such as constructor parameters, get/set methods, property bags, etc.
 * **Visualizing a model** to see a graphical representation of the code-based model.
 * **Simple type conversions** such as string => xml.
 * **Spatial data types** such as SQL Server's `geography` & `geometry`.
 * **Many-to-many relationships** without join entity. You can already [model a many-to-many relationship with a join entity](https://docs.efproject.net/en/latest/modeling/relationships.html#many-to-many).
 * **Alternate inheritance mapping patterns** for relational databases, such as table per type (TPT) and table per concrete type TPC.

* CRUD
 * **Seed data** allows a set of data to be easily upserted.
 * **ETag-style concurrency token support** 
 * **Eager loading rules** allow a default set of related data to always be retrieved when an entity is queried.
 * **Filtered loading** allows a subset of related entities to be loaded.
 * **Simple command interception** provides an easy way to read/write commands before/after they are sent to the database.

* Providers
 * **Azure Table Storage**
 * **Redis**
 * **Other non-relational databases**

* Platforms
 * **Universal Windows Platform (UWP)** currently works for local development but there are issues with the .NET Native tool chain that the EF and .NET Native teams are working to address.
 * **Xamarin** works in some scenarios but has not been fully tested as a supported scenario.

## EF Core 1.2

EF Core 1.2 is currently in the planning phase.

## EF Core 1.1 (Released)

The following features are already implemented and included in the 1.1 release.

* **Explicit Loading** allows you to trigger population of a navigation property on an entity that was previously loaded from the database.
* **DbSet.Find** provides an easy way to fetch an entity based on its primary key value.
* **Connection resiliency** automatically retries failed database commands. This is especially useful when connection to SQL Azure, where transient failures are common.
* **EntityEntry APIs from EF6.x** such as `Reload`, `GetModifiedProperties`, `GetDatabaseValues` etc.
* **Field mapping** allows you to configure a backing field for a property. This can be useful for read-only properties, or data that has Get/Set methods rather than a property.
* **Memory-Optimized Tables** are a feature of SQL Server. You can specify that the table an entity is mapped to is memory-optimized. When using EF Core to create and maintain a database based on your model (either with migrations or `Database.EnsureCreated()`), a memory-optimized table will be created for these entities. 
* **Simplified service replacement** makes it easier to replace internal services that EF uses.

## EF Core 1.0 (Released)

The following features are already implemented and included in the 1.0 release.

* Modelling
 * **Basic modelling** based on POCO entities with get/set properties. The common property types from the BCL are supported (`int`, `string`, etc.).
 * **Built-in conventions** that build an initial model based on the shape of the entity classes.
 * **Fluent API** allows you to override the `OnModelCreating` method on your context to further configure the model that was discovered by convention.
 * **Data annotations** are attributes that can be added to your entity classes/properties and will influence the EF model (i.e. adding [Required] will let EF know that a property is required).
 * **TPH inheritance pattern** allows entities in an inheritance hierarchy to be saved to a single table using a discriminator column to identify they entity type for a given record in the database.
 * **Relationships** between entities based on navigation and foreign key properties.
 * **Shadow state properties** (properties that are part of the model but do not have a corresponding property in the CLR class).
 * **Alternate keys** and the ability to define a relationship that targets an alternate key.
 * **Model validation** that detects invalid patterns in the model and provides helpful error messages.
 * **Key value generation** including client-side generation and database generation.
 * **Relational: Table mapping** allows entities to be mapped to tables/columns.
		
* Change Tracking
 * **Snapshot change tracking** based on recording the original values of an entity when it is retrieved from the database.
 * **Notification change tracking** allows your entities to notify the change tracker when property values are modified.
 * **Accessing tracked state** of entities (via `DbContext.Entry` and `DbContext.ChangeTracker`).
 * **Attaching detached entities/graphs**. The new `DbContext.AttachGraph` API helps re-attach entities to a context in order to save new/modified entities.
		
* SaveChanges
 * **Basic save functionality** allows changes to entity instances to be persisted to the database.
 * **Optimistic Concurrency** protects against overwriting changes made by another user since data was fetched from the database.
 * **Async SaveChanges** can free up the current thread to process other requests while the database processes the commands issued from `SaveChanges`.
 * **Transactions** means that `SaveChanges` is always atomic (meaning it either completely succeeds, or no changes are made to the database). There are also transaction related APIs to allow sharing transactions between context instances etc.
 * **Relational: Batching of statements** provides better performance by batching up multiple INSERT/UPDATE/DELETE commands into a single roundtrip to the database.
		
* Query
 * **Basic LINQ support** provides the ability to use LINQ to retrieve data from the database.
 * **Mixed client/server evaluation** enables queries to contain logic that cannot be evaluated in the database, and must therefore be evaluated after the data is retrieved into memory.
 * **NoTracking** queries enables quicker query execution when the context does not need to monitor for changes to the entity instances (i.e. the results are read-only).
 * **Eager loading** provides the `Include` and `ThenInclude` methods to identify related data that should also be fetched when querying.
 * **Async query** can free up the current thread to process other requests while the database processes the query.
 * **Raw SQL queries** provides the `DbSet.FromSql` method to use raw SQL queries to fetch data. These queries can also be composed on using LINQ.

* Database schema management 		
 * **Database creation/deletion APIs** are mostly designed for testing where you want to quickly create/delete the database without using migrations.
 * **Relational database migrations** allow a relational database schema to evolve overtime as your model changes.
 * **Reverse engineer from database** scaffolds an EF model based on an existing relational database schema.

* Database providers
 * **EntityFramework.SqlServer** connects to Microsoft SQL Server 2008 onwards.
 * **EntityFramework.Sqlite** connects to a SQLite 3 database.
 * **EntityFramework.InMemory** is designed to easily enable testing without connecting to a real database.
 * **3rd party providers** are available for other database engines. See [Database Providers](https://docs.microsoft.com/en-us/ef/core/providers/) for a complete list.

* Platforms
 * **Full .NET** includes Console, WPF, WinForms, ASP.NET 4, etc.
 * **.NET Core (including ASP.NET Core)** targeting both Full.NET and .NET Core on Windows, OSX, and Linux.

