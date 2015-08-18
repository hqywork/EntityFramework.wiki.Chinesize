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
		
* Change Tracking
 * **Snapshot change tracking** based on recording the original values of an entity when it is retrieved from the database.
 * **Accessing tracked state** of entities (via `DbContext.Entry` and `DbContext.ChangeTracker`).
 * **Attaching detached entities/graphs**. The new `DbContext.AttachGraph` API helps re-attach entities to a context in order to save new/modified entities.
		
* SaveChanges
 * Basic save functionality
 * Optimistic Concurrency
 * Async SaveChanges
 * Relational: Transactions
 * Relational: Table-based CUD
 * Relational: Batching of CUD statements
		
* Query
 * Basic LINQ support
 * Mixed client/server evaluation
 * NoTracking
 * Eager loading (`Include`)
 * Async query
 * Translation of common BCL functions
 * Raw SQL queries (with composability)

* Database schema management 		
 * Database creation/deletion APIs (mostly for testing)
 * Database Error Page
 * Relational database migrations 

* Database providers
 * In-Memory
 * SQL Server
 * SQLite

* Platforms
 * Full .NET
 * ASP.NET 5
 * Universal Windows Platform
	
### In Progress

The following features are currently being implemented. Some scenarios may work, but there are significant limitations as the work is incomplete.

* TPH inheritance pattern
 * This has been implemented in parts of the stack, but still needs to be implemented in ModelBuilder, conventions, and migrations.

* Cross-cutting quality
 * Docs (same model as ASP.NET 5)
 * IntelliSense docs
 * API Reviews

* Query
 * Navigation properties in LINQ

* Reverse engineer from database


* Platforms
 * Mac/Linux Support

* Data Stores
 * Postgres (delivered by npgsql)
 * SQL Compact (delivered by ErikEJ)

* Data annotations

For RTM
* Design time context discovery/loading
* Deployment (blocked by Tooling)
* Performance
 * Add additional coverage
 * Improvements based on results
* Cascade delete (scoped)
* Logging DNX min-bar

### Backlog Features

