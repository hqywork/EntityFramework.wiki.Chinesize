# Entity Framework 7 (EF7)

Below is the schedule and roadmap for EF7. Please note that these dates and feature plans are all subject to change. As with any project of this size it is difficult to predict exactly when things will land. Even so, we think it's important to be as open and transparent as possible about our plans so that our users can have the right expectations and create their plans accordingly.

## Schedule

The schedule for the initial release of EF7 is guided by the [ASP.NET 5 schedule](https://github.com/aspnet/Home/wiki/Roadmap). While EF7 is not strictly tied to ASP.NET 5 (and has many use cases outside of ASP.NET), it is an integral part of ASP.NET 5 and it is therefore important that we have a stable release of EF7 to support the ASP.NET 5 release.

|Milestone|Release Date|
|---------|------------|
|Beta7    | 31 Aug 2015|
|Beta8    | 21 Sep 2015|
|RC1      |    Nov 2015|
|7.0.0    |     <span title="Calendar based">Q1<sup>*</sup></span> 2016|

The November release candidate (RC1) will be a supported and production ready cross-platform release. Depending on feedback from RC1 we will ship additional release candidates as necessary.

## Features

Because EF7 is a new code base, the presence of a feature in past releases does not mean that the feature is implemented in EF7. For this reason, we have provided a list of what is implemented and what we plan to implement before the initial release of EF7. 

We have also provided a list of features that we think are important but will not be enabled in the initial release. This means that EF6.x continues to be the best version of EF for a number of applications until these features are implemented in the EF7 code base.

### 7.0.0 Features

#### Implemented

The following features are already implemented and included in the last official pre-release. Note that features on this list may still have bugs that need to be resolved and the APIs/design may still change as we progress toward the first stable release.

* Modelling
 * **Basic modelling** based on POCO entities with get/set properties. The common property types from the BCL are supported (int, string, etc.).
 * **Relationships** between entities based on navigation and foreign key properties.
 * **Shadow state properties** (properties that are part of the model but do not have a corresponding property in the CLR class).
 * **Unique constraints and indexes** (and the ability to define a relationship that targets a unique constraint that is not the primary key).
 * **Built-in conventions** that build an initial model based on the shape of the entity classes.
 * **Model validation** that detects invalid patterns in the model and provides helpful error messages.
 * **Key value generation** including client-side generation and database generation.
 * **Relational: Table mapping** allows entities to be mapped to tables/columns.
		
* Change Tracking
 * **Snapshot change tracking** based on recording the original values of an entity when it is retrieved from the database.
 * **Accessing tracked state** of entities (via `DbContext.Entry` and `DbContext.ChangeTracker`).
 * **Attaching detached entities/graphs**. The new `DbContext.AttachGraph` API helps re-attach entities to a context in order to save new/modified entities.
		
* SaveChanges
 * **Basic save functionality** allows changes to entity instances to be persisted to the database.
 * **Optimistic Concurrency** protects against overwriting changes made by another user since data was fetched from the database.
 * **Async SaveChanges** can free up the current thread to process other requests while the database processes the commands issued from `SaveChanges`.
 * **Relational: Transactions** means that `SaveChanges` is always atomic (meaning it either completely succeeds, or no changes are made to the database). There are also transaction related APIs to allow sharing transactions between context instances etc.
 * **Relational: Batching of statements** provides better performance by batching up multiple INSERT/UPDATE/DELETE commands into a single roundtrip to the database.
		
* Query
 * **Basic LINQ support** provides the ability to use LINQ to retrieve data from the database.
 * **Mixed client/server evaluation** enables queries to contain logic that can not be evaluated in the database, and must therefore be evaluated after the data is retrieved into memory.
 * **NoTracking** queries enables quicker query execution when the context does not need to monitor for changes to the entity instances (i.e. the results are read-only).
 * **Eager loading** provides the `Include` and `ThenInclude` methods to identify related data that should also be fetched when querying.
 * **Async query** can free up the current thread to process other requests while the database processes the query.
 * **Translation of common BCL functions** enables these functions to be translated into database specific query language (i.e. SQL) when they are used in LINQ.
 * **Raw SQL queries** provides the `DbSet.FromSql` method to use raw SQL queries to fetch data. These queries can also be composed on using LINQ.

* Database schema management 		
 * **Database creation/deletion APIs** are mostly designed for testing where you want to quickly create/delete the database without using migrations.
 * **Database error page** is a piece of middleware for ASP.NET 5 that provides additional help for database related exceptions.
 * **Relational database migrations** allow a relational database schema to evolve overtime as your model changes.

* Database providers
 * **EntityFramework.SqlServer** connects to Microsoft SQL Server 2008 onwards.
 * **EntityFramework.Sqlite** connects to a SQLite 3 database.
 * **EntityFramework.InMemory** is designed to easily enable testing without connecting to a real database.

* Platforms
 * **Full .NET** includes Console, WPF, WinForms, ASP.NET 4, etc.
 * **ASP.NET 5** targeting both Full.NET and .NET Core.
 * **Universal Windows Platform (UWP)** applications can make use of the SQLite provider to access local data
	
#### In Progress

The following features are currently being implemented. Some scenarios may work, but there are significant limitations as the work is incomplete.

* Modeling
 * **Data annotations** are attributes that can be added to your entity classes/properties and will influence the EF model (i.e. adding [Required] will let EF know that a property is required).
 * **TPH inheritance pattern** allows entities in an inheritance hierarchy to be saved to a single table using a discriminator column to identify they entity type for a given record in the database.

* Cross-cutting quality
 * **Documentation** is being developed in the [EntityFramework.Docs](https://github.com/aspnet/EntityFramework.Docs) repository.
 * **IntelliSense documentation** allow contextual help within Visual Studio when using the EF APIs.
 * **API reviews** involve us going over each API to ensure we have a clean and consistent API surface.

* Performance
 * **Additional coverage** is being added to our benchmark suite.
 * **Performance improvements** to address the identified bottlenecks are ongoing.

* Query
 * **Navigation property translation** allows dotting thru navigation properties in LINQ (e.g. `Products.Where(p => p.Category.Name == "Food")`)

* **Reverse engineer from database** scaffolds an EF model based on an existing relational database schema.

* Platforms
 * **Mac and Linux** currently work for EF7 but we are improving stability.

* Data Stores
 * **Postgres** support is being developed by [Npgsql](https://github.com/npgsql/npgsql)
 * **SQL Compact** support is being developed by [ErikEJ](https://github.com/ErikEJ/EntityFramework7.SqlServerCompact)

#### Scheduled for 7.0.0

The following features are on our list to be implemented prior to the 7.0.0 release, but are not currently being actively worked on.

* **Design time context discovery/loading** allows tooling (such as migrations) to correctly locate your context type and instantiate it to create the model and identify the database it connects to.
* **Deployment** provides better support for deploying database changes as part of your general application deployment.
* **Cascade delete** enables automatic deletion of child records when the parent is deleted.
* **Logging** for the initial release will be simplistic and conform to the suggested guidelines for ASP.NET 5. We will improve our logging story in subsequent updates.

### Backlog Features

#### Critical O/RM features

The things we think we need before we say EF7 is the recommended version of EF. Until we implement these features EF7 will be a valid option for many applications, especially on platforms such as UWP where EF6.x does not work, but for many applications the lack of these features will make EF6.x a better option.

* Query
 * Explicit Loading
 * Sub queries
 * Group by translation to SQL

* Logging
 * Great logging story (polish/consistency etc.)
* Logging++ (structured logging for Glimpse etc.)

* Update model from database

* Modelling
 * Complex/value types

* Change Tracking
 * Missing APIs from EF6.x (`Reload`, `GetModifiedProperties`, etc.)
 * Entry methods for relationships
 * Entry methods for database values

* Relational specific
 * Sproc-based CUD
 * Connection resiliency

#### High priority features

There are many features on our backlog and this is by no means an exhaustive list. These features are high priority but we think EF7 would be a compelling release for the vast majority of applications without them

* Modelling
 * Shadow state entities
 * Mapping to methods, alternate property patterns, property bags, immutable objects, etc.
 * Visualize a model
 * Composable functions support  
 * Custom conventions
 * Entity/Table splitting
 * Simple type conversions (i.e. string => xml)
 * many:many relationships without join entity

* Change Tracking
 * Notification change tracking

* CRUD
 * Seed data
 * Lazy loading (feedback based)
 * Simple ETag-style concurrency token support 
 * Eager loading improvements, e.g. rule-based, aggregate-based, filtered, for derived classes, etc.
 * Simple interception mechanisms for query and updates

* Providers
 * ATS
 * Redis
 * Other non-relational databases

* Migrations
 * CLI (non-DNX projects)

* Provider specified in config file