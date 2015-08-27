# Graph behaviors

## Add/Attach for graphs

See [Issue #2726](https://github.com/aspnet/EntityFramework/issues/2726) for discussion of the issue.

Conclusions:
- There should be Add/Attach/Update methods that work with more than just the single entity provided
- These methods will use metadata to determine how much of the given graph to add or attach
  - We considered adding metadata to define aggregates and may still do this in the future, but for now we will simply add/attach all dependents, but will not traverse to principals
  - So Adding a category will also add all the related products. But if a product has a reference to a supplier, then this will not be added. Similarly, if a product is added, then its category will not be added.
  - In essence, each entity that roots a graph of dependents should be added/attached appropriately
- If we are attaching, but the entity has a null/sentinel key, then we will add it instead. This allows, for example, a category with some existing and some new products to be attached, and if the new products do not have keys set, then they will be correctly added.
- DetectChanges will use the same rules to bring in added dependents. It follows that DetectChanges will not bring in principals.
- There will still be methods that work only with the single object
  - Changing the state of an entity will only effect that object
  - Also, the Add/Attach/Update methods will have a flag to make them single object methods
  - We also considered multiple method names, but the general feeling was that a flag with a default would result in cleaner, more intuitive API surface

## Relationship snapshot

There was an idea that the relationship snapshot, which is used to detect changes in navigation properties, could be avoided if information in the tracked graph could be used to reconstruct it. However, for this to work the entire graph reachable from an entity (up to some well-defined stopping point) must always be tracked. This then as behavior implications, notably:
- DetectChanges could bring in objects and Add them if it is called while half way through painting state on a graph
- Detaching an entity would result in the entity being brought back in by DetectChanges unless its relationships are severed or it is somehow marked as being tracked as detached. :-)

For these reasons we decided to continue to not require that an entire graph (or subgraph) is tracked and we will keep the relationship snapshot.

### Cascade delete by convention

We discussed whether cascading delete behavior should be switched on by convention for non-nullable FK relationships. On the pro side:
- The old stack did this and we did not see complaints
- If cascading delete is not defined and the principal is deleted, then it will result in an exception--in other words, there isn't a counterpoint behavior that doesn't throw
- It would follow the principle used by other conventions--that is, setting up reasonable semantics by convention so that explicit configuration to avoid throwing is not required

On the cons side:
- If an application doesn't want to do cascade delete, and the developer is not aware that it is on by convention, 
and an application bug does something that would cause a cascade delete, then data will be deleted instead of the application throwing an exception (and likely crashing)
- Sometimes it can result in an invalid cyle on SQL Server, as it did with the old stack, but this is a SQL Server problem and is easily avoided by explicit configuration

We decided to stay with the behavior of the old stack and switch on cascade deletes by convention.

# Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/2946) to provide feedback, ask questions, etc.
