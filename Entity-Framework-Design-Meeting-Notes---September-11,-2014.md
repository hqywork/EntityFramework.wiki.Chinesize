# EF Design Meeting Notes - September 11, 2014

## Migrations commands (high-level experience)

### Ways to use Migrations in EF7

The core functionality for Migrations in EF7 lives in the EntityFramework.Migrations library. This functionality can be used in a variety of ways--for example:
- As PowerShell commands, where supported, just like in previous versions of EF
- Through an executable tool, similar to migrate.exe in previous versions of ef
- Through Project K (aspnet.vnext) commands, where supported
- Potentially through Visual Studio tooling, although this work is currently not scheduled
- Through third-party tools, if anybody decides to create one

This discussion primarily concerns the PowerShell and Project K commands. The Project K commands allow essentially the same functionality as the PowerShell commands. The difference is that they work in the Project K environment and hence will work with and use this infrastructure, such as the json-based project system. It also means they will run on platforms where the PowerShell commands are not available, such as on Linux.

### Using Migrations in Project K

The project.json file will need a package dependency and a command. For example:

```
{
    "dependencies": {
        "EntityFramework.SQLite": "7.0.0-*",
        "EntityFramework.Commands": "7.0.0-*"
    },
    "commands": {
        "ef": "EntityFramework.Commands"
    }
}
```

It is our hope that the project.json can be updated automatically to include these things, perhaps as part of package install. As of now these things need to be added manually.

The package name in git is currently EntityFramework.Design. However, we discussed other names and settled on EntityFramework.Commands since this code is used for the PowerShell, Project K, and executable commands. The .Design posfix makes it seem like System.Data.Entity.Design or similar packages/assemblies, which is conceptually closer to the EntityFramework.Migrations package. We also considered EntityFramework.Tools, but this would likely cause confusion with Visual Studio tooling. Note that while we are discussing Migrations here the actual package will likely also include non-Migrations commands.

Once setup, to add a migration use something like:

```
k ef migration add InitialCreate
```

This will create a new folder (called "Migrations" by convention, but can be configured) if needed and add the scaffolded migrations and model snapshot. Note that Initialize-Migrations is not required because Migrations in EF7 does not have its own MigrationsConfiguration class. Migrations are initialized the automatically the first time a migration is added.

The migration can be applied and the database updated using:

```
k ef migration apply
```

`k ef help` will display the EF magical unicorn ASCII art and a list of available commands.

### Planned PowerShell and K commands

| NuGet PMC | K |
|-----------|---|
| (Tab expansion on -Project) | dir .. |
| (Select default project) | cd |
| (Tab expansion on -Context) | k ef context list |
| Use-DbContext [Context] -Project | k ef context use [context] |
| Add-Migration [Name] -Context -Project | k ef migration add [name] -context |
| (Tab expansion on -Migration) | k ef migration list -context |
| Update-Database [Migration] -Context -Project | k ef migration apply [migration] -context |
| Script-Migration [From] [To] -Idempotent -Context | k ef migration script [from] [to] -idempotent -context -Project |

Notes:
* We want to have some consistency in terms between the PowerShell commands, the APIs in EntityFramework.Migrations.dll, and the K commands. However, we also want to fit with the conventions for commands in each environment. Therefore we are trying to use the same terms in each place, but organized appropriately for the environment--e.g. "Add-Migration" in PowerShell, "migration add" in K. UpdateDatabae in the PowerShell commands is interesting because it is not doesn't fit with the general pattern of Migrations commands. We are therefore proposing to make the EF7 PowerShell command Apply-Migration. Update-Database will still be available and functional but will be obsoleted and will issue a warning with a note to use Apply-Migration instead.
* We discussed whether the commands should be "k ef migration blah" or  "k ef migrations blah". We settled on the singular for now since it is consistent with the PowerShell commands and other command line tools such as those for Azure management.
* We discussed dropping "ef" from the command, but decided to leave it in as a consistent single place to keep all ef commands whether or not they are directly related to Migrations.
* We discussed whether or not "using" a DbContext should be persistent or not. There are pros and cons to both--for a single context project persistence seems useful; for a multiple context project persistence may be annoying. We did not come to a firm decision in the meeting.

### Package layout

The proposed EntityFramework.Commands NuGet package layout is:

* build/
  * net45/;win81/;wpa81/
    * EntityFramework.Commands.targets (copies EntityFramework.Commands.dll to bin/)
* lib/
  * aspnet50/;aspnetcore50/
    * EntityFramework.Commands.dll (has k commands)
  * portable-net451+win81+wpa81/
    * _._ (allows install)
* tools/
  * EntityFramework.psm1
  * EntityFramework.Commands.dll (has AppDomain stuff)
  * ef.exe

This layout implies the following:
* The Migrations commands will be available in some form on all platforms that EF7 is available. The two assemblies are actually different but compiled from the same source--the K assembly supports K but not PowerShell; the assembly for other targets supports PowerShell.
* For non-K projects the commands assembly is not deployed with the application by default since it lives only in tools and not in lib. Note that this does not mean that Migrations functionality is not deployed, but rather than the PowerShell/K commands are not deployed.
* Currently the commands assembly will be deployed when using Project K. This should change when K supports design-time dependencies. K ignores the "tools" part of the package.

## Update pipeline for batching

An overview of the update pipeline for EF7 was covered in a [previous design meeting](https://github.com/aspnet/EntityFramework/wiki/Entity-Framework-Design-Meeting-Notes---July-10,-2014#update-pipeline-overview). The update pipeline was recently updated to support batching of commands. This is a basic overview for how this works. Note that this discussion does not include the generated SQL, which will vary based on provider and other considerations, but rather covers how the update pipeline creates and handles batches.

As previously discussed the update pipeline performs a topological sort to produce a list of sets. The sets must be executed in order. The commands in each set are not completely sorted and may be sorted as desired by the provider for efficiency, to reduce the chances of deadlock, or for any other reason that the provider has.

Given the sorted list of commands in each set, the update pipeline attempts to add each command to the batch. No more commands will be added to the batch if:
* The maximum number of parameters allowed would be exceeded
* The command text generated would be too long
* Whatever other provider-specific limitation is reached

Once a batch of commands has been generated it will be executed. Once executed and store-generated values are immediately propagated back to entities such that these values can be used directly in subsequent batches. (Note that the store-generated values will be discarded if the transaction fails.)

Analysis indicates that, at least one SQL Server and other databases we have looked at, the parameter limit is likely to be reached long before the command gets too long. Therefore, the code is optimized to be fast for splitting batches on parameter limit as opposed to splitting on command text being too long. There are also heuristics for determining that command text is likely to be too long when the next command is added. Both of these things mean that it is unlikely that the command text will ever actually get too long. If it does get too long, then some processing may need to be repeated, but this seems better than making the common case slower to avoid this unlikely outcome.

The current code does not reuse parameters with the same values. This could be a useful future optimization.

## Metadata for keys, foreign keys, and indexes

Currently each entity type has ordered sets of columns for:

* Keys
  * Assumed to always have unique values
  * One is marked as the primary key and provides default identity
  * Used to define the properties at the principal end of relationships
* Foreign keys
  * Can be marked as unique
  * Used to define the properties at the dependent end of relationships 
  * Reference a principal key to fully define the relationship
* Indexes
  * Can be marked as unique
  * Can be annotated with additional provider-specific index information
  * Used to define an index in the store for any desired reason

The discussion was around whether or not all of these need to be distinct top-level concepts. Consider:
* Having a key or a foreign key usually implies an index, but this index should be in some way explicitly specified. This is because we had issues in early versions of Migrations where indexes were entirely inferred from keys/foreign keys. This lead to problems when the indexes in the database did not necessarily match the inferred indexes. Therefore, the model should in some form explicitly specify which indexes exist.
* Specifying the index for a key or foreign key could be a simple annotation on the key or foreign key metadata item, as opposed to being an entirely separate item in metadata
* Note that even though indexes are often associated with keys or foreign keys it is still common to have indexes where no key or foreign key is needed
  * However, it could be that having a key in the metadata model is okay even if it is not needed, so we could merge indexes and keys. This would imply that we support non-unique keys, for which the semantics in the state manager are currently not clear.
* Should these be part of core metadata or moved into relational annotations?
* Should there be a common base class?

Decision:
* We will keep the three object types and have an explicit index objects for every index, even if it is associated with a key or foreign key. This keeps the metadata model for indexes clean and simple
  * We will consider composition patterns or references between items to make explicit the relationship between a key or foreign key and its associated index, and also to potentially reduce the overhead of saving column information twice.
* We will not create a common base class at this time. There don't seem to be cases where it is useful to treat these things polymorphically and the semantics of a common base class are also questionable.
* We will keep these concepts in core metadata because they are useful concepts to reason about at the core level. For example, parts of the system like the state manager and client-side query evaluation may be able to reason about indexes for improved performance. (Foreign keys (or, in less relational terms, common value associations) are in the model because they are a pragmatic way of storing relationship information co-located with entities and useful for shredding and transmitting graphs to other tiers. Foreign keys may often be in shadow state so that they do not appear in the domain model.)


