# Graph behaviors

## Goals

* Add relatively simple support for attaching graphs that can be done for RTM
* Try not to block future plans for higher-level services in this area
* Be more of a pit-of-success for attaching disconnected graphs than we have in the old stack

## Background

The old stack always attached the entire graph of reachable entities in the same state. That is, if you called Attach, then everything reachable would be put in the Unchanged state. When attaching a graph where some entities already existed and some were new this meant that:
* The graph was forced through an "invalid" state where entities where marked in a state that they would not ultimately end up as. For Attach this invalid state was often cause an exception due to primary key conflicts on objects that did not yet have a key and therefore all had the default value for the CLR type--e.g. zero.
* Even if Add was used to avoid the exception from Attach, the application code was still forced to do a graph traversal to fixup the invalid state.
* There were no built-in mechanisms to determine what state new entities should be put in, so this had to be written by the application.

Ultimately, we want to support the following strategies for determining the state of new entities:
* Compare with existing graph, either containing full or partial information
  * This graph could be obtained by querying the database--i.e. the Merge behavior
  * Or the graph of state info could be obtained from information travelling with the entities through the client round-trip
* Use primary key values—new objects don’t have a PK set yet, so they should be Added, while other entities should be Unchanged (or possibly Modified)
* Attach aggregate root and entire aggregate is attached with given state

## Proposal

The main part of the proposal is to introduce a new method

`void AttachGraph(object root, Action<EntityEntry> callback)`

This method starts at the root entity and uses graph traversal to find other entities in the graph. For each entity the callback delegate is invoked which can set the state of the entity as appropriate. The callback delegate could also do other things such as setting original values or adding shadow state.

Notes:
* Only entities that are not already being tracked will get the callback. That is, graph traversal in a particular direction will stop as soon as a tracked entity is encountered.
* Internally, this method will likely use a general purpose graph iterator for the model, which will be made publicly available.
* We considered returning a list of the entries that were attached, but decided this was YAGNI for now. It is not a re-compile breaking change to change void into something else later.
* We discussed if this method should go on the context, the set, both the context and set, or the change tracker. We decided on the change tracker.
* We will not add a params overload for multiple roots at this time.
* There will be an Async version because key generation can require a database access.

An additional part of the proposal is to introduce a simple version of AttachGraph:

`void AttachGraph(object root)`

This method would use a callback that looks at the primary key value to determine whether an entity should be Added or Unchanged.

Notes:
* It should be possible to get access to this callback such that it can be used together with other actions such as setting shadow state, or so that it can be used to mark objects as Modified instead of Unchanged.
* We should look at integration with DI such that the behavior of this method can be changed by service replacement.
* This strategy will only work when entities that should be Added do not have key values assigned and are making use of EF key generation. The method will throw if it encounters an entity that does not have key generation enabled and has no key value assigned.

We also discussed removing the single object Add/Attach/Update/Remove methods and replacing with:

`EntityEntry<TEntity> Attach<TEntity>(TEntity entity, EntityState state)`

This is because each of these four methods does exactly the same thing just with a different state. We decided not to do this as it was felt the existing methods would be easier to understand and preserves some level of experience from EF6. Also, this method is effectively the same as setting the state with `context.Entry(Foo).State = newState`. However we decided that given setting state can now be a significant amount of work and may be async we will make the State property read-only and add SetState and SetStateAsync methods.

# DbSet Queries

## Background

Currently DbSets are IQueryables and using them results in a database query. However, they also have Add, Remove, etc. and so act as a way to introduce entities into the unit of work, etc. But since the query is still a database query it means that if those new entities are not returned when the set is enumerated. This is usually unintuitive to new users. For example:

```
using(var db = new BloggingContext())
{
    var blog = new Blog();
    db.Blogs.Add(blog);
    Assert.True(db.Blogs.Contains(blog)); // Fails as database doesn't yet include the new blog
}
```

On the other hand, most people seem to like this syntax for writing database queries and do find it intuitive:

```
using(var db = new BloggingContext())
{
    var blogs = db.Blogs.ToList();
}
```

So having the DbSet work as a database query is not universally bad.

## Options

One option which would be perhaps the most intuitive is to make DbSet queries integrate database results with local results and return the union of both. We would like to do this, but we don't have time to do it now. Given that we can't do this now, other options are:

* Keep the current current implementation
  * This is not intuitive to new users in the case where entities have been added to the set
  * This is consistent for users of previous versions of EF
  * The simple use of DbSet properties for database queries is preserved
* Make DbSet not work for queries at all and add properties/methods for different queries
  * DbSet.DatabaseQuery, DbSet.LocalQuery, etc.
  * DbSet would not be IQueryable/IEnumerable at all, and the simple database query pattern we have now would fail to compile
  * This option as the advantage that the type of query is explicit and discoverable, and new types of query can be added in the future.
  * The DbSet could maybe be made some form of query in a later release.
* The same as the previous option, except that DbSet acts as either a local or combined query.
  * As mentioned above, we do not have time to do the combined query at this time.
  * The problem with making it a local query is that simple code that would previously return results from the database will now do nothing, which will be very confusing.
* Remove Add/Remove from DbSet, or create a new type that represents just a database query and does not allow adding/removing entities.

We decided to go with the first option and keep the current semantics. This is because:
* Changing DbSet to not be a query at all will result in people having to figure out the new pattern and use it, and this new pattern is not much better for common database query cases.
* Changing DbSet to be just a local query will be even more confusing because people will expect database results and instead get nothing.
* Making DbSet return the combined results is scoped out at this time.

Hopefully in the future we can implement the combined query semantics and then enable this with a flag on the context or through a new DbSet type.

# Migrations and namespaces

Recent changes mean that when a new migration is added it will be placed into the same namespace as the previous namespace for that context. There was agreement that this was good, but the following tweaks were suggested:
* Currently, the namespace is used to determine the folder location. No attempt is made to look at the file system. This is fine when namespaces match file system structure, but will result in people doing manual work to move scaffolded migrations when the file system and namespace to not align. To fix this, we will look for a file with the expected name in the project structure and, if found, use that as the folder location for the new migration. If no such file is found the namespace will be used as it is now.
* Currently migrations for a context are put in the Migrations folder/namespace. If multiple contexts are used then migrations for the new context must be manually moved. Instead, if migrations for a second context are scaffolded they will be put into a Migrations/ContextName folder/namespace. Note that this may require fully-qualifying the context name in the attribute to avoid conflicts.

# Discussion

Please use the design meeting [discussion issue](https://github.com/aspnet/EntityFramework/issues/1248) to provide feedback, ask questions, etc.