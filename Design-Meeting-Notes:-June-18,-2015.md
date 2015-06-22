# Target Frameworks

(Presented by [Brice](https://github.com/bricelam))

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

Content coming soon...

# Table rebuilds in Migrations

(Presented by [Nate](https://github.com/natemcmaster))

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
