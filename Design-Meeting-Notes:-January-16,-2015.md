# API Review #1 (DbContext, DbSet, Database)

## DbContext

### Constructors

- [ ] Remove constructor that takes service provider AND options. This isn't needed because the options can just be registered in the service provider.
   - Update: We decided to keep this constructor because it is hard to share a service provider but use different EF providers when not using a derived context type unless this constructor is present.

### Configuration

- [X] Move `AutoDetectChangesEnabled` to `DbContext.ChangeTracker` and remove the `Configuration` property. Future settings can be grouped where they belong and we won't have a general settings/configuration drill down.

### Add/Attach/Update/Remove

- [X] Add an `IEnumerable` overload to avoid needing to call `ToArray` all the time when calling with multiple entities. 
- [X] Swap the return type of collection overloads to be `void` to avoid needing to create `EntityEntry` for each entity (you can always loop over the supplied entities and get the entry if needed). In the future we could return an `IEnumerable` that lazily creates the entries if needed.
- Update: We also changed the name of the multiple entity methods to AddRange, etc. to better handle overload resolution.

### AddAsync

- [ ] Remove `params` overloads and just have `IEnumerable` (`AddAsync` is not going to be used very much and the `params` prevents us having an options `CancellationToken` which results in lots of overloads).
   - Update: we decided to remove AddAsync entirely

### ScopedServiceProvider

- [X] Introduce an `IAccessor<TService>` interface and expose this as explicit implementation of `IAccessor<ServiceProvider>`

## DbSet<TEntity>

### Protected Constructor

Currently provided to allow creating test doubles, but can be confusing because you have to use the other ctor to create a functional DbSet.

- [X] Make `DbSet<TEntity>` an abstract base class with default ctor that has no functionality in it. This achieves the same thing as an interface but allows breaking changes.
- [X] Make what's currently implemented in `DbSet<TEntity>` a derived class with just the `DbContext` ctor and all the functionality.
- [X] `DbContext.Set` will still be typed as `DbSet<TEntity>` but creates instances of the derived type.


### Add/Attach/Update/Remove/AddAsync

- [X] Same changes as DbContext methods

### Queryable/Enumerable methods

- [X] Ensure these interfaces are explicitly implemented so they don't show up in IntelliSense.

### ServiceProvider

- [X] Expose an explicit implementation of `IAccessor<ServiceProvider>` to support extension methods.

## Database

### Protected Constructor

- [X] Remove this constructor as it's not clear that mocking this class is really a valid scenario.

### Exposing dependencies

Remove these properties which are exposed to support extension methods
- [X] DataStoreCreator
- [X] Logger
- [X] Model


***

- [X] Remove IDatabaseInternals and replace with the following explicit interface implementations:
- [X] `IAccessor<DataStoreCreator>`
- [X] `IAccessor<ILogger>`
- [X] `IAccessor<IModel>`
- [X] `IAccessor<ServiceProvider>`