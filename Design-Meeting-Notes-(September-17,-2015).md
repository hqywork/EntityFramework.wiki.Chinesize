# Design-time services

At design-time (i.e. for the Migrations and Reverse Engineer tooling) there are several different sources for services that need to work together.

* Runtime services (e.g. `Migrator`)
* EF design-time services (e.g. `MigrationsScaffolder`)
* Provider design-time services (e.g. `SqliteMetadataModelProvider`)
* User design-time services (e.g. `MyMigrationsCodeGenerator`)

At design-time, we'll use the following algorithm to load the runtime services.

1. Look in **startupProject** for the `Startup` class
2. Call `Startup.ConfigureServices`
3. Look in services for the `DbContext`
4. If none, look for `DbContext` elsewhere (see below)
5. Get runtime services from the `DbContext`
6. Add provider's design-time services (see below)
7. If no `Startup` class was found in step 1, look in **project**
8. Call `Startup.ConfigureDesignTimeServices` to add user's design-time services

For operations that don't require a `DbContext` (e.g. Reverse Engineering), the steps can be simplified as follow.

1. Look in **startupProject** for the `Startup` class
2. Call `Startup.ConfigureServices` to add runtime services
6. Add provider's design-time services (see below)
8. Call `Startup.ConfigureDesignTimeServices` to add user's design-time services

Here is the algorithm for finding and instantiating a `DbContext` that couldn't be found from `Startup.ConfigureServices`.

1. Look in **startupProject** for an `IDbContextFactory<DbContext>`
2. If none, Look in **project** for an `IDbContextFactory<DbContext>`
3. If found, return `IDbContextFactory<DbContext>.Create`
4. Look in **startupProject** for the `DbContext`
5. If none, look in **project** for the `DbContext`
6. If none, look in **project** for `Migration`s, look at their referenced `DbContext`
7. Return `Activator.CreateInstance(typeof(DbContext))`

To add the provider's design-time services, here are the steps.

1. Look in provider assembly for `[assembly: DesignTimeProviderServices]`
2. Load the assembly and type referenced by the attribute
3. Call `ConfigureDesignTimeServices` on the type

# Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/3159) to provide feedback, ask questions, etc.
