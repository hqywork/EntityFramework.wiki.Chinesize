The purpose of this page is to list out the things that help us make decisions when designing/implementing/reviewing EF7 (and later releases). This will help us make individual decisions that result in a cohesive stack.

**NOTE**
> These are principles and not hard-and-fast rules. Time constraints, user experience, or other circumstances may mean we violate the principles.

## High Level Principles
The following principles define what we are trying to achieve with EF7. They shape how we prioritize and design features.
 
**EF7 is a major evolution of EF. It's about building the right stack for the future. We want to be considerate of our past without being constrained by it.**
* EF7 is a breaking change release. When there is a better pattern/API/etc. we take the change (unless the benefit is trivial).
* EF7 will not replace EF6 on the day we RTM 7.0.0.
* EF6 will still be the best choice for many applications for some time.
* We are not going to push folks to upgrade, keeping existing application on EF6 is valid and supported.
* Moving from EF6 to EF7 is not an "upgrade" scenario, it's a "port" scenario.
* EF7 is not a re-implementation of EF6 (EF7 has simpler mapping capabilities, no EDM, etc.).

**EF7 prioritizes features based on their individual merit rather than EF6 parity.**
* EF7 uses the API names from EF6 unless there is a compelling reason to change them.
* If a method/property/type does pretty much the same thing then it has the same name (unless the old name was really bad) 
* Names that app developers don't generally type out (i.e. Fluent API return types, low level building blocks, etc.) do not need to stay the same.
 
**EF7 fits into the vision/messaging around Cross-platform .NET, .NET Core, ASP.NET 5, etc.**
* We shouldn't be coming up with messaging around cross-platform/lightweight/modular/re-write at the EF level.
* We're just part of the larger picture of what's happening in .NET at Microsoft.

**EF7 runs everywhere that people write .NET code **
* The core runs everywhere, but not all providers will run on every platform.
* EF7 supports the default provider(s) for each platform (i.e. SQL Server for Windows, SQLite for devices, Postgres/MySql for Mac/Linux).

**EF7 supports new data stores (relational and non-relational)**
* EF7 is a great O/RM. We don't want to sacrifice the relational experience in order to light up new data stores.
* EF7 allows O/RM concepts/patterns to be used with non-relational data stores.
* EF7 is not trying to replace SDKs for non-relational stores. It is an option for folks who want the O/RM like patterns.
* EF7 does not hide the type of database you are targeting. We are not building a magical abstraction where the same model can transparently target different types of providers. We provide a common set of building blocks but there are provider specific APIs (e.g. configuring row/partition key on Azure Table Storage, beginning a DbTransaction on a relational database).

**EF7 is optimized around being a simple, intuitive O/RM**
* EF7 is not a micro-O/RM (it still supports LINQ, change tracking, updates, etc.)
* EF7 favors an opinionated "this is how you do things" approach over "we support any mapping pattern".
* EF7 allows you to map the same database schemas as past releases of EF, but your domain objects may not be able to drift as much from the shape of your schema.
* EF7 allows you to easily circumvent O/RM features when you need to get close to the metal (e.g. drop down to SQL when LINQ doesn't work).
 
**EF7 is lightweight and composable.**
* EF7 will be built with memory and CPU usage in mind
* EF7 will be built to avoid unnecessary abstraction/layering between top level APIs and the database (i.e. "closer to the metal" that EF6)
* EF7 is pay-per-play (i.e. if you don't use SQL Server/commands/etc. then you don't download, reference, deploy, or load the binaries)
* EF7 is built over composable building blocks (i.e. you can override/replace individual components, drop down to low level metadata model, etc.) 
 
## Constraints
The following constrains define the scope for where EF7 needs to work well. They allow us to design features that work well within the defined scope and don't have to be complicated or degraded by supporting scenarios outside the scope. They allow us to not invest time fixing bugs that are outside the defined scope.
 
**SQL Server**
* EF7 supports SQL Server 2008 onwards. Things need to work well by default with 2012. It's ok to have to change settings etc. to work on 2008.
* EF7 works well by-default on SQL Azure (i.e. without changing settings etc.).
 
**Platforms**
* EF7 supports .NET 4.5.1 onwards
* EF7 doesn't have to support older versions of each platform (i.e. Win10 UWP support is more important than Win81, Mono 4 over Mono 3.x, etc.).
 
## API Conventions
The following conventions help us have a consistent API across our product. We also encourage provider writers to follow the same conventions.
 
**Prefixes**
* We don't have a global prefix that we use everywhere (in EF6 we think we overdid the "Db" prefixing)
* We use the "Db" prefix where there is a need to qualify something to EF (usually when it is a very general concept such as "Model")
* Types that implement a base type, interface, etc. from Core (i.e. the relational implementation of DataStore) use the same namespace as the type they implement and append a library specific prefix (i.e. RelationalDataStore).
 * Example prefixes:
 * Relational
 * SqlServer
 * Sqlite
 * InMemory
 
**Postfixes**
* When applying the a postfix we do not shorten the name of the component it relates to (i.e. DbModel/DbModelBuilder is correct and DbModel/ModelBuilder would be wrong)
 * Common postfixes:
 * Factory = a component that creates a new instance of a particular type
 * Source = a component that locates the correct instance of a particular type
 * Cache = a component that creates or selects a component based on its presence in a cache
 * Builder = a component that provides an API for constructing an instance of another type. The type being constructed may or may not be immutable.
 
**Visibility**
* By default everything is public (see Namespaces for info on categorizing things that are "Internal")
* If we have helper types that are strictly an implementation detail, have no foreseeable use to external code, and are ugly to have in public API we may choose to make them internal (these should be very limited).
 
**Namespaces**
* Our root namespace (`Microsoft.Data.Entity`) is only for types that users directly reference in their code for the majority of applications (i.e. `DbContext`, `DbSet`, etc.)
* We group related components into a sub-namespace if there are enough types (~5 types) or we think there will be in the future.
* Components that don't have a grouping go into the `Infrastructure` namespace
* We use an `Internal` sub-namespace to identify components that would historically have been internal types.
 * We do not guarantee that these APIs will not break/change/etc. between releases (including minor/patch releases)
 * We do not document these types
 
**Extension Methods**
* Extension methods typically go in the namespace of the type they target so that they are easily discovered without importing namespaces
* Exceptions to this are for very common types where you may have EF referenced but the extension methods don't make sense in all places you use the target type, or the extensions methods have a generic name that may collide with other extension methods (e.g. `IQuerable<T>.ToListAsync` lives in `System.Data.Entity` rather than `System.Linq`)
