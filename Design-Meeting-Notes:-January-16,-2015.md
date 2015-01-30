# API Review #1 (DbContext, DbSet, Database)

## DbContext

### Constructors

- [ ] Remove constructor that takes service provider AND options. This isn't needed because the options can just be registered in the service provider.

### Configuration

- [X] Move `AutoDetectChangesEnabled` to `DbContext.ChangeTracker` and remove the `Configuration` property. Future settings can be grouped where they belong and we won't have a general settings/configuration drill down.

### Add/Attach/Update/Remove

- [ ] Add an `IEnumerable` overload to avoid needing to call `ToArray` all the time when calling with multiple entities. 
- [ ] Swap the return type of collection overloads to be `void` to avoid needing to create `EntityEntry` for each entity (you can always loop over the supplied entities and get the entry if needed). In the future we could return an `IEnumerable` that lazily creates the entries if needed.

### AddAsync

- [ ] Remove `params` overloads and just have `IEnumerable` (`AddAsync` is not going to be used very much and the `params` prevents us having an options `CancellationToken` which results in lots of overloads).

### ScopedServiceProvider

Introduce an `IAccessor<TService>` interface and expose this as explicit implementation of `IAccessor<ServiceProvider>`

## DbSet<TEntity>

### Protected Constructor

Currently provided to allow creating test doubles, but can be confusing because you have to use the other ctor to create a functional DbSet.

- [ ] Make `DbSet<TEntity>` an abstract base class with default ctor that has no functionality in it. This achieves the same thing as an interface but allows breaking changes.
- [ ] Make what's currently implemented in `DbSet<TEntity>` a derived class with just the `DbContext` ctor and all the functionality.
- [ ] `DbContext.Set` will still be typed as `DbSet<TEntity>` but creates instances of the derived type.


### Add/Attach/Update/Remove/AddAsync

- [ ] Same changes as DbContext methods

### Queryable/Enumerable methods

- [ ] Ensure these interfaces are explicitly implemented so they don't show up in IntelliSense.

### ServiceProvider

- [ ] Expose an explicit implementation of `IAccessor<ServiceProvider>` to support extension methods.

## Database

### Protected Constructor

- [ ] Remove this constructor as it's not clear that mocking this class is really a valid scenario.

### Exposing dependencies

Remove these properties which are exposed to support extension methods
- [ ] DataStoreCreator
- [ ] Logger
- [ ] Model


***

- [ ] Remove IDatabaseInternals and replace with the following explicit interface implementations:
- [ ] `IAccessor<DataStoreCreator>`
- [ ] `IAccessor<ILogger>`
- [ ] `IAccessor<IModel>`
- [ ] `IAccessor<ServiceProvider>`