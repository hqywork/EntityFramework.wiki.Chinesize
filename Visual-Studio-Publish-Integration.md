# EF7 and Visual Studio Publish

In Visual Studio you can **right-click -> Publish...** on an project to walk thru a deployment wizard. This wizard should help you configure/deploy all aspects of your application, including databases. There are two aspects of database deployment that apply to Entity Framework.
* **Configure a connection string** to be used in the deployed application that differs from the connection string used during development.
* **Apply any pending migrations** for models that use Migrations to maintain the database.

## Configure a connection string

### What we did in EF6

In EF6 we used `DbContextInfo` to determine if adding/updating a connection string in `web.config` would successfully change the connection for the context. If so, we would provide the option to override the connection string during deployment and update the config file with the string provided.

The logic we used in EF6 was complex and required creating an instance of the context to find out where it's connection string came from. This was also problematic if you had a non-default constructor or you had startup code that was required to setup the environment (since we would just load the assembly and try to create an instance of the context). In EF7 and ASP.NET 5 we want to have a simpler and more opinionated approach.

### What we're doing in EF7

Detect any types that derive from `DbContext` in the project being deployed (or it's project references) and provide the option to override the connection string.

In EF7 we've introduced an `EntityFramework` section in `config.json` that is formatted as follows. We will only allow overriding the connection string for contexts whose connection is present in this section of configuration. We will transform the config file to have the new connection string (or override it in some other manner). If we find a `DbContext` that does not have a corresponding connection string in config we can display a message informing developers how to enable connection string replacement during deployment.

```
"entityFramework": {
    "<derived context type name>": {
        "connectionString": "<connection string>"
    }
}
```

We also need to handle the following syntax in `config.json`. Given the developer is giving us the actual connection string to use, we can just replace the `connectionStringKey` with a `connectionString` value. 

```
"entityFramework": {
    "<derived context type name>": {
        "connectionStringKey": "<key of another config value>"
    }
}
```

## Apply any pending migrations

When migrations are used to maintain the database schema, you should be able to easily apply them to the target database as part of deployment. For EF models that target an existing database there is no such requirement, as the schema is managed externally to EF.

### What we did in EF6

In EF6 we detected any types that inherited from `DbContext` and provided options for deployment.  If the context had migrations enabled we would provide a checkbox to apply migrations on app start. This was achieved by registering the `MigrateDatabaseToLatestVersion` initializer for the given context in `web.config`. This meant any pending migrations would be applied during the first operation that used the context on the deployed site (typically the first page request that access data).

#### The issues with EF6 approach

This approach is flawed for a number of reasons:
* Failures during the migration process do not result in failed deployment. They are only seen during the first page request that accesses the database. Given that error details are typically not displayed in a production application, it is difficult to debug.
* Applications deployed to multiple app servers could result in two app servers attempting to migrate the database at the same time.
* Some migrations can take a significant amount of time to apply and these would block any data access while applied.
	
#### Other options that won't work

There are other options we considered that also don't work:
* Running migrations from deployment machine is not always possible because the deployment machine may not be able to connect to the database server (this is true by default in SQL Azure). We provide components that can be used to achieve this for automated deployment scenarios when it is possible, but it's not something we can use for this publish wizard.
* Back in EF4.3 days we investigated creating an MSDeploy provider that could apply the migrations from the app server. The issues is that MSDeploy tasks are executed with elevated permissions and executing migrations involves running user code. Therefore, this is unacceptable from a security perspective.
	
### What we're doing in EF7

The flow for EF7 and ASP.NET 5 will be:

1. Detect any types that derive from `DbContext` in the project being deployed (or it's project references).
1. Search all projects in the solution to see if there are any types that derive from `Migration` and have a `[ContextType]` attribute that references the derived context type. 
 * Searching all projects is required because there is no requirement for the migrations project to be referenced by the application (or a library that contains the context).
1. If there aren't any migrations, provide the user with an informational message that deployment cannot apply migrations or create the database. This will typically only be the case when EF is targeting an existing database that isn't maintained by EF.
1. If there are migrations, provide a checkbox to apply migrations during deployment (and the ability to preview the script to be used)
 * Use the migrations command line tool to create a script that will apply migrations (`k ef migration script`). 
 * The command is run on the project that contains the migrations and specifies the project being deployed as the startup project.
 * This script is 'idempotent' in that it will use the `__MigrationsHistory` table to ensure that each migration is only applied once. This means we can calculate a script locally and it is safe to apply to the remote database regardless of its current state. 
 * Note the command doesn't currently produce the idempotent script (tracked by issue #1280).
 * Note there is currently no way to specify the startup project (tracked by issue #823).
1. Use the dbFullSql MSDeploy provider to execute the script as part of deployment

## Limitations

There are some assumptions/limitations of this approach:
* We generate the script locally so your app would need to be targeting the same database provider as when deployed. There is currently no way to change the provider via `config.json` so this seems reasonable for the moment. When we enable this in the future, we may need to apply any overrides to the provider when generating the script.
* The context and migrations must be defined in projects within the solution.
* The migrations project must have the **EntityFramework.Commands** package installed and have it registered under commands in `project.json` as the `ef` command.
* If the connection string is defined in `project.json` but is then not used or overridden by code the connection string override will not apply.