# Entity Framework Design Meeting Notes - July 10, 2014

## Update pipeline overview

The basic flow of the relational update pipeline is:
- The state manager determines which entities have changes and passes these to the relational-specific data store SaveChanges method.
- The entities are processed into commands where the commands are the individual insert, update, and deletes that need to happen.
- These commands form a graph where the vertices are commands and the edges are dependencies between commands. For example, when inserting a principal and a dependent the principal needs to be inserted first. Likewise, when deleting a principal and dependent, the dependent needs to be deleted first.
- A Topological sort is used to create a list of sets of commands. The list is sorted into the order that the commands should be executed. The sets are partially sorted. A provider can choose to do further sorting on the sets if necessary or for optimizations.
- The commands are batched. Currently there is only ever one command per batch, but this will change as batching is implemented.
- Up until this point there is no use of ADO.NET specific objects such as DbCommands, etc.
- The batches are executed.
- If the commands will result in store-generated values, then these are read back by the executor and propagated into the state manager.
- Optimistic concurrency is checked by making sure that the correct number of rows were affected.
- Note that creating the ADO.NET commands in a batch, executing them and reading the results is handled by a single object.

## Value generation

EF supports automatic generation of values for properties. This is most often used for generation of primary key values, but can be used for the generation of any property value. Depending on the strategy being used several different areas in the code may get involved to make the overall strategy work. These areas are:
- Generation of values when new entities are first tracked
  - This is important because in EF7 even newly added objects must have keys
  - Sometimes the keys generated at this stage are temporary and replaced when the entity is saved
- Generation of values when entities are saved
  - For example, a database with an Identity column will generate a value when a new entity is inserted
- Handling of database defaults
  - When to include the value in the insert command and when to leave it out to let the database generate a default
- Determining whether an entity should be treated as a new entity or an existing entity
  - It is common to treat entities with null/default keys as new
  - This will be discussed more at a later date

On top of this there are higher-level experiences that need to be realized:
 - The top-level API for common behaviors
 - The behavior to use by convention
 - For both these things, how abstract/provider-agnostic should the experience be?
   - Can the model and annotations be the same for the same basic experiences even if the strategy used by different providers is different?

We will focus on the lower-level design at this time and come back to the top-level experiences later.

### Example strategies

Using relational stores and integer keys as an example, three of the common strategies that we want to support are described below with details of what happens when a new entity is added and what happens when it is saved.
 
 - Integer key mapped to Identity (or equivalent) column
   - Add: Temporary value generated unless real value exists already
   - Save when value is temporary: Use store-generated real value
   - Save when value is real: On SQL Server requires turning Identity Insert on/off, which we may not support since the strategy below using a sequence and a column default is cleaner
 - Integer key mapped to column with sequence default
   - Add: Temporary value generated unless real value exists already
   - Save when value is temporary: Use store-generated real value
   - Save when value is real: Just use real value
 - Integer key using store sequence on Add
   - Add: Real value generated unless real value exists already
   - Save: Use real value
   - Note that this pattern involves extra round-trips to the database to get key values when the entity is Added, but it allows the real key values to be used immediately which can be useful in some app scenarios and has significant advantages for batching. Also, the additional round-trips are minimized by requesting blocks of values to use rather than requesting each value separately.

Similar patterns also exist for other types--most notably GUIDs.

### Conclusions

Questions and conclusions from these strategies:
- How should EF determine if the property value has already been set when Add is called so as to know whether or not to generate a value?
  - First, the property must be marked in metadata as needing value generation. If this is not done then no value generation will happen regardless of the property value.
  - If the property is marked appropriately then the current value will be compared to a sentinel. The default sentinel will be the CLR default for the property type. If the sentinel is matched, then value generation will happen, otherwise it won't.
- How does the update pipeline know what kind of value has been generated on Add?
  - Is it a real value or a temporary value?
  - For some types it is possible to imagine a good key-space for temporary values--e.g. negative numbers. For other types, e.g. GUIDs, there is no natural and obvious partitioning of the space.
  - However, the value generator used on Add knows if it is generating temporary values or not. It can therefore set a flag in the state entry (similar to the IsModified flag) indicating that the value is temporary.
  - This also means that it is possible for app code to set temporary values explicitly by marking them as such in the state entry.
- How does the update pipeline deal with default values?
  - For relational stores, when a property value is marked as temporary it will not be included in the insert command.
- What about computed values? That is, properties for which a value is generated on insert and update.
  - These will be marked as read-only be default. The update pipeline will ignore read-only values and so will not include the values in the update command. However, a computed value can be made read-write, in which case it will be included only if it has been modified. The store will need to be set up in a way that ensures that this is allowed.
