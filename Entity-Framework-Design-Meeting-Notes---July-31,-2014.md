# EF Design Meeting Notes - July 31, 2014

## Value generation - high level experience

The low-level aspects of value generation were discussed in an [earlier design meeting](https://github.com/aspnet/EntityFramework/wiki/Entity-Framework-Design-Meeting-Notes---July-10,-2014). In this meeting we discussed
* Top-level API for common behaviors
* Behavior to use by convention
* The degree of abstraction/provider-coupling for the APIs and conventions

### Abstract API proposal

The proposal is to be very abstract and provider-agnostic at the highest level. This consists of:
* Marking a property simply as needing value generation, without specifying how this will happen
* Each provider will define a default strategy to handle this
 * This strategy must use a value generator to make sure that an appropriate value us generated on Add
 * Make sure that the update pipeline (e.g. SQL generation/command handling) can handle saving with value generation by the database if needed
* If a provider can't support value generation at all for the given property, then it should throw

An advantage of this behavior is that it maps pretty well to the existing DatabaseGenerated attribute in the .NET Framework, which is hard to change because it is in the .NET Framework.

### Provider specialization

Providers can provide fluent API (or new attributes) to change the default strategy. Such API/attributes will cause the model to be annotated appropriately so that value generation on Add/Save work correctly together.

We considered adding API at the relational level for all relational providers and we may still do this later, but for now it does not seem worthwhile.

### SQL Server value generation

There will likely be three in-the-box strategies for SQL Server:
* Traditional Identity column
 * Generate a temporary value on Add
 * Update pipeline will retrieve Identity value on Insert
* Column making use of database Sequence
 * Generate a temporary value on Add unless a specific value is already set
 * Update pipeline will retrieve the value obtained from the Sequence on Insert, unless a specific value was already set
* Obtain a value from database Sequence on Add
 * Blocks of values obtained from the database sequence will be used to supply values on Add, unless a specific value was already set
 * Update pipeline does nothing special since value already generated

We decided to try the final strategy as the default for now because:
* Batching in SaveChanges becomes significantly cleaner and has less overhead
* Actual key values are generated immediately rather than going through a temp key stage

However, we will need to do perf testing to ensure that the occasional additional round trip does not negate the improvements in batching. Also, it may make it harder when mapping to existing databases or database created with older versions of EF since these are more likely to have Identity columns.

Note that while most of the infrastructure code for these strategies exists there are still some things to be worked out:
* Integration with Migrations
 * This means that Migrations needs to see that a Sequence is used and create the Sequence object in the database if it does not exist
 * It may not be desirable to delete this object when going down because this could cause the Sequence to be reset, but then it should be deleted if the database is migrated to empty and it is desirable to allow it to be deleted at any time
 * Should Migrations create on Sequence per database by default, or one Sequence per table, or one Sequence per column?
  * We will start with one Sequence per database but make it easy to specify use of a different Sequence per column. (It is uncommon to have more than one column in table use a Sequence so one per column/one per table is the same in most cases.)

### Relationship to default values

As discussed previously:
* Property can be marked as read-only. This means it will never be included in updates.
 * “Computed” properties will be read-only by default
* Property value can be marked as temporary. This means it will not be included in updates.
 * Allows value generators to mark values as temporary for update pipeline consumption
 * Can also be used by other code--for example, black/white listing

Is there a need for any other metadata around default values? Currently it doesn't seem like we need anything else, but as with everything we may revisit this down the road.

## Code First conventions

The EF7 model is significantly less complicated than the EDM model used in previous versions of EF. It is also available and mutable immediately that model building begins. This means that the approach used for conventions in previous versions of Code First is likely not appropriate in EF7. These are some high-level thoughts on how conventions might work in EF7.

### Old stack Code First characteristics

From EF4.1 to EF6 Code First interacted with conventions in the following ways
* Fluent configuration overrides data annotations, which overrides conventions
* Fluent API allows partially configured model
 * For example, can specify FK even if PK is not yet known
 * This is important to minimize the code needed to override just one convention. For example, it should be possible to specify an FK that is not found by convention without also having to specify the PK that is/will be discovered by convention
* Actual model is not built until all information has been gathered
 * Instead we build up configuration models to record information and then use these to build the EDM model after all information has been gathered

### EF7 conventions: Option 1

Apply conventions immediately when an item enters the model:
* For example, conventions run when property is discovered/added/changed
  * Conventions listen to the types of items they are interested in
  * Beginnings of this already in the stack
* Pros: one model, always as up-to-date as it can be
* Cons: Hard to preserve precedence order
 * Model must support partial configuration, or some restrictions on call order
 * Convention might overwrite explicit configuration. For example, PK discovered -> FK set explicitly -> PK set -> FK overwritten
 * Or convention might not change configuration by previous convention based on new information. For example, PK discovered -> FK set by convention -> PK set -> FK left in bad state
 * Without additional information switching behaviors to overwrite/not overwrite will flip between failure in the first or second examples above

### EF7 conventions: Option 2

As option 1, but record where model changes come from
* Handles precedence order problem
  * Convention won’t overwrite explicit configuration. For example, PK discovered -> FK set explicitly -> PK set -> FK not changed since set explicitly
  * Convention can change configuration by previous convention based on new information. For example, PK discovered -> FK set by convention -> PK set -> FK changed based on new info
* Potentially makes data annotation precedence easier since we can record that a change was made due to data annotation

This is the option we will try initially. Concerns to keep in mind:
* Running conventions reactively might have significant perf impacts. We should consider allowing running conventions to be temporarily disabled. Also, conventions should be careful to only register for events that they are actually interested in.
* What is the implications of this approach for the "lightweight" conventions API?
 * It may be that the conventions API really becomes bulk configuration more than conventions. This is much easier to implement in EF7 where the real model actually exists and is being kept up-to-date during model building.

### EF7 conventions: Option 3

Run all conventions after building the model with ModelBuilder is complete:
* Pros: All explicit configuration is already done, so can’t overwrite
  * Although still need to know difference between set and not set--for example, if property is set as needing no value generation then convention should not set value generation, which is different from property set as needing no value generation because it is just the default value of the annotation
* Cons:
  * Model must handle partial configuration
  * Model only partially configured until conventions run. For example, can’t ask for/use PK when building model if it is discovered by convention
  * Conventions still have to be written carefully to avoid overwriting, so we may still need to know if something was set by convention or not

### EF7 conventions: Option 4

Build configuration model, as for EF6.
* Pros: Less impact on “real” model
  * Doesn’t need to support partial configuration
  * Don’t need to record where a change came from
* Cons:
  * More data structures, more complexity
  * Real model is not available until the end. For example, can't really look at and use the real model while building it, which can be very useful.

We decided to try option 2.

## Code First relationship API

We [previously discussed](https://github.com/aspnet/EntityFramework/wiki/Entity-Framework-Design-Meeting-Notes-June-5,-2014#fluent-api) the Code First fluent API for EF7 and came up with some changes. However, on reflection the proposed API may not be better:
* Similar in some respects to old API, but using terms Collection/Reference
  * These are very general terms
  * It is not immediately obvious that one is the inverse of the other
* 1:1 API requires an FK
  * Some people believe that it is a bad idea to always require the FK to be specified in the fluent API for normal bidirectional relationships even when it does not exist in the domain model
  * How do you specify a unidirectional 1:1 relationship?
  * How about a relationship with no navigations? Presumably this is through passing something like IsUnique to the FK, but then there is one pattern for bidirectional relationships and a different one for relationships with one or zero navigations.

## New proposal

The proposal is to model the API on how we talk about relationships. For example, for one-to-many bidirectional relationship:
```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToMany(d => d.Posts, p => p.Blog);
});
```

Note that the multiplicity of the relationship is immediately obvious. Also, the entire relationship is specified in one call. This hopefully removes some of the confusion around under-specified relationships being 1:* by default, and also side-steps the need to attempt to connect up relationships specified partially from either side.

For one-to-many unidirectional relationship:
```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToMany(d => d.Posts);
});
```

Note that the same method is called, just without specifying one of the navigations.

For one-to-many relationship with no navigations:
```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToMany<Post>();
});
```
Note that again the same method is called. In this case the generic type must be specified since it can not be inferred from the navigation property expression.

In each case further methods can be chained to modify the relationship or provide additional specification. For example:
```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToMany(d => d.Posts, p => p.Blog)
        .ForeignKey(d => d.BlogId)
        .Key(d => d.Id)
        .Required();
});
```

Note that these are the same regardless of whether the relationship as zero, one, or two navigations.

The same pattern can be applied for one-to-many relationships specified from the other side. For example:
```
modelBuilder.Entity<Post>(b =>
{
    b.ManyToOne(p => p.Blog, d => d.Posts)
        .ForeignKey(d => d.BlogId)
        .Key(d => d.Id)
        .Required();
});
```

Also, one-to-one relationships use essentially the same pattern:
```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToOne(p => p.Owner, d => d.Blog);
});
```

```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToOne(p => p.Owner);
});
```

```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToOne<BlogOwner>();
});
```

```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToOne(p => p.Owner, d => d.Blog)
        .ForeignKey<BlogOwner>(d => d.BlogId)
        .Key<Blog>(d => d.Id)
        .Required();
});
```

Note that specifying either the foreign key or the principal key (with Key) disambiguates which end is the principal and which is the dependent. In the first three examples the convention is that the entity that is used to do the configuration is the principal.

The generic type must be specified for the ForeignKey or Key calls since it cannot be inferred/constrained.

We could also allow specification of the principal/dependent end without needing to reference the PK/FK at all just by telling us which side the principal key is located. For example:
```
modelBuilder.Entity<Blog>(b =>
{
    b.OneToOne(p => p.Owner, d => d.Blog)
        .Key<Blog>();
});
```

We expect the many-to-many API to be very similar to the one-to-many and one-to-one APIs.

We discussed moving the specification of relationships up to the model builder rather than hanging it off the entity type configuration. However, it was felt that having the entity type to start with and provide generic type information was useful and would be preferred by most.

We also discussed using a single name (e.g. "Relationship" or "Association") with overloads for 1:*, 1:1, etc. However, this complicates the cases with zero or one navigation properties where overload resolution may not be effective. Also, it may be easier to understand/read when the multiplicity is explicit in the code.

### Second proposal

The second proposal was to keep basically the same API as in EF6 but change some names and move the Required out of the navigation property specification to avoid the IsRequiredPrincipal and IsRequiredDependent confusion. For example:
```
modelBuilder.Entity<Blog>(b =>
{
    b.Many(e => e.Posts)
        .One(e => e.Blog)
        .ForeignKey(e => e.BlogId)
        .Key(e => e.Id)
        .Required();
});
```

```
modelBuilder.Entity<Blog>(b =>
{
    b.One(e => e.Owner)
        .One(e => e.Blog)
        .ForeignKey<BlogOwner>(e => e.BlogId);
});
```

```
modelBuilder.Entity<Blog>(b =>
{
    b.One<BlogOwner>()
        .One<Blog>()
        .ForeignKey<BlogOwner>(e => e.BlogId);
});
```

While this seems better than what we currently have we decided to go with the first new proposal for now.
