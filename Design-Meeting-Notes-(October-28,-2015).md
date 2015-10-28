DNX Commands and Startup Projects
=================================
The original design of the DNX commands was that you would invoke it on the target project (e.g. MyApp.Data) and pass in the startup project (e.g. MyApp) using the `--startupProject`. However, DNX is not able to load an assembly that isn't referenced by the current project, and referencing it would result in a circular dependency since the reference needs to go the other direction (e.g. MyApp needs to reference MyApp.Data).

In order to fix this, we're swapping the two projects. The commands will now need to be invoked on the ***startup***
project, and a `--targetProject` option will be provided.

For example, to add a migration you would use the following commands.

```cmd
cd src\MyApp
dnx ef migrations add MyMigration --targetProject MyApp.Data
```

This will add a new migration to the MyApp.Data project using MyApp as the startup project.

Discussion
==========
Please use [the discussion issue](https://github.com/aspnet/EntityFramework/issues/3590) to provide feedback, ask questions, etc.