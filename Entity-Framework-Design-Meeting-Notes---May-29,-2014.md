# Entity Framework Design Meeting Notes - May 29, 2014

## Runtime database creation in EF7

Previous versions of EF supported database initializers that would execute the first time a context was used and were typically used for creating and seeding new databases. These were implemented in EF 4.1 as a means to use Code First before the Migrations feature was available.

In EF7 we will have Migrations from the start and we intend to guide people to use Migrations for most database creation scenarios. We do not intend to support automatic initializers since they added a lot of complexity and hampered understanding of what was going on due to the high level of magic.

However, we currently still believe that there is value in being able to create databases at runtime without directly using Migrations. The main scenarios for his are testing and rapid prototyping, although it could also be useful for automatically creating a database on first run when working with devices. (For this last scenario it may be better to scaffold a migration, deploy it with the app, and then have the ability to easily run this migration on the device when the app is initializing. Of course, when using SQLite the database file itself could be deployed for new apps.)

Given that we want to have some form of runtime database creation there are several questions that arise.

### Create MigrationHistory table?

Should the database created contain a populated __MigrationHistory table as would be the case if Migrations had been used? 

Creating the table potentially makes it easier to switch to Migrations at a later time. However, to do this safely and automatically we would need to store the model in the __MigrationHistory table. This is fragile and something that we don't need for other Migrations scenarios in EF7, so we currently don't plan to do it.

Also, it is not uncommon to want to switch to using Migrations for a database that was created in some other way and therefore does not have a __MigrationHistory table.

Therefore, the current decision is to not create the __MigrationHistory table in this situation. Instead we plan to provide more discoverable commands/tooling for using Migrations with an existing database. These commands/tooling would likely support the patterns described here: http://msdn.microsoft.com/en-us/library/dn579398.aspx

### MVC application templates

We want to have a good, easy experience when getting started with creating a new MVC application with Identity in Visual Studio. However, using runtime database creation for this can result in it being used in production apps, which is usually not a good idea. (For example, it makes evolving the app harder and doesn't handle multiple instances/requests attempting to create the same database at the same time.) Therefore we want to guide people to use Migrations for this.

The tentative plan here is to:
- Have a pre-scaffolded migration for the Identity database in the project template.
- The template code together with help from Identity and EF should determine that the database does not exist. This might be through catching a specific exception type thrown by Identity or EF.
- At this point some kind of page would be shown indicating that the database needs to be created. Ideally this page would have a button to run the scaffolded Migration to create the database. This button could be part of a Migrations Dashboard which provides a UI for running and managing Migrations, appropriately secured to the admin user. A similar UI could also be provided for use in Visual Studio when developing.

### Abstraction of the API

Currently the top-level runtime database creation APIs in EF7 are:
 - Database.Create
 - Database.CreateTables
 - Database.Delete
 - Database.Exists
 - Database.HasTables
 - Database.EnsureCreated
 - Database.EnsureDeleted

The first five of these are simple operations while the Ensure... methods are composed from the simple operations. The Ensure... methods cover the common pattern for database creation in testing and rapid prototyping. They can also be reasonably implemented across multiple different types of data store.

The simple methods are not so easy to abstract in a meaningful way beyond relational databases. For example, knowing that a database exists but has no tables is useful in relational providers since it is common for an empty database to be provisioned for which EnsureCreated will detect that it is empty (HasTables returns false) and then appropriately create the tables. But for other types of store it may not be possible to check whether or not the database has tables. Therefore we will move the simple operations to be relational-only and leave only EnsureCreated and EnsureDeleted on the common surface.

