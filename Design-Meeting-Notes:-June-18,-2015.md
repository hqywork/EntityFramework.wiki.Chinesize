# Target Frameworks

The `dotnet` target framework moniker was recently introduced, and Entity Framework was updated to use it. For more information about this new target see [Oren's blog post](http://oren.codes/2015/06/16/demystifying-pcls-net-core-dnx-and-uwp-redux/). This prompted a review all of the frameworks we target and why. Here are the details.

Framework | When      | Why
--------- | --------- | ---
net45     | Always    | For Mono; To throw on ambient Transactions
dnx451    | As needed | For DNX-specific APIs; To workaround [aspnet/dnx#2031](https://github.com/aspnet/dnx/issues/2031)
dotnet    | Always    | For Windows 10, .NET Native, CoreCLR, DNX Core & all future frameworks
dnxcore50 | As needed | For DNX-specific APIs
netcore50 | As needed | For Windows 10 design-time

We also intend to target Xamarin, but we're currently limited by the .NET Core packages.

If Mono updates their `System.Data` API, we'll consider dropping our `net45` target. If the DNX minimum framework version becomes 4.6 or the .NET Core packages support 4.5.1, we can leverage `dotnet` in more places.

# Default SQL Server value generation strategy

For the past few Beta's the default key generation strategy for SQL Server has been a HiLo pattern using a sequence. We discussed changing this because:
* The default version of SQL Azure databases (v11) does not support sequences.
* The feedback we are seeing suggests folks expect `IDENTITY` and tend to swap back to that rather than considering the benefits of HiLo based on a sequence.

Our decision is to swap back to using `IDENTITY` by default for numeric primary keys. Developers can easily opt-in to sequence based HiLo key generation using the (**soon to be renamed**) `modelBuilder.ForSqlServer().UseSequence()` API.

This discussion was just around what we do by default. The following patterns will be supported by EF7.
* `IDENTITY` key columns.
* Key value generated during `INSERT` using a default value.
* Key generation during `Add`, where key values are fetched from the database. We support HiLo with a sequence by default but folks could also use custom key generators to do this with a key table etc.
* True client side key generation (probably most appropriate for `GUID`s).

# Table rebuilds in Migrations

We plan to implement better migration support for SQLite. Some DDL operations, such as ALTER COLUMN or DROP COLUMN, are not supported by SQLite. The migration pipeline for SQLite will attempt to identify these cases and provide the appropriate workaround for SQLite's limitations. If no workaround is suitable, migrations will fail rather than corrupting data.

For example, we will support dropping columns in SQLite. A simple migration might look this in code:
```c#
public class DropLeaderColumnMigration : Migration
{
   public override void Up(MigrationBuilder builder)
   {
       builder.DropColumn(name: "leader", table: "countries");
   }
   ...
}
```

SQLite does not support `DROP COLUMN` natively, but the same result can be accomplished with a table-rebuild. EF will translate the DropColumn command from the migration into the appropriate SQL statements. In this example, the following statements will drop the "leader" column from the "countries" table:
```sql
ALTER TABLE countries RENAME TO countries_temp;

CREATE TABLE countries (
	name TEXT PRIMARY KEY,
	population NUMBER);

INSERT INTO countries (name,
 population)
SELECT name, population FROM 
countries_temp;

DROP TABLE countries_temp;
```


# Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/2439) to provide feedback, ask questions, etc.
