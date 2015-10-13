Provider API for Reverse Engineering
==================

For the RC1 release, we will attempt to define an API for provider-writers that will enable them to use reverse engineering against their datastore. Provider writers can choose to implement this API at one of two levels.

1. Provider writers can implement an interface responsible for returning an `IModel`. This should represent completely the entities, properties, and navigation properties to be passed into code generation.
2. Providers can use an abstract class provided in the framework which will take a simple domain model about database schema and build the `IModel` for the provider.

The exact names of these interfaces have not yet been decided, but they will be put into the `Microsoft.Data.Entity.Scaffolding` namespace before RC1. 

To watch progress on these update, follow these issues: [#3329](https://github.com/aspnet/EntityFramework/issues/3329), [#3335](https://github.com/aspnet/EntityFramework/pull/3335), and [2956](https://github.com/aspnet/EntityFramework/issues/2956).