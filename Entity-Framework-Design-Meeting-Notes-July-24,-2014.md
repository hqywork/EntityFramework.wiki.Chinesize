# EF Design Meeting Notes July 24, 2014

## Code First fluent API: closures and chaining

When designing the Code First fluent API for EF7 we try to ensure that it has a range of desirable characteristics. Some of these that are relevant to this discussion are, in no particular order:
* Simple overload/Intellisense experience
 * Have few overloads to choose from
 * Be able to choose which one to use
 * Be able to figure out how to use it
* Don’t repeat yourself (DRY)
 * E.g. should not need to call Entity multiple times for same type
* Be consistent
* Be easy to copy/paste fluent code from EF6 applications
* Be able to address any metadata item with extension method
  For example, must be able to address a Key to name it for relational

### Method chaining

Consider In this code, which is essentially the way things are in EF6:

```
modelBuilder
    .Entity<Customer>()
    .Key(e => new { e.Id, e.Name })
    .Property(e => e.Id)
    .ColumnName("ID");
```

The Key method returns the EntityBuilder so that other calls to configure the entity type can be chained after the Key call--in this case, the Property call. However, notice that the only way to configure additional information about the key using chaining is to introduce new overloads (or equivalent) of Key because the Key method returns an EntityBuilder rather than a KeyBuilder.

Conversely, the Property method returns the PropertyBuilder. This allows additional configuration of the property through chained calls--for example, the ColumnName call in the code above. However, if more configuration of the entity type is needed, then a new Entity() call is needed because there is no way to get back to the EntityBuilder once Property has been called.

### Nested closure

The nested closure pattern is a way to get around the limitations of chaining. For example:

```
modelBuilder
    .Entity<Customer>()
    .Key(kb => 
         {
            kb.Properties(e => new { e.Id, e.Name })
                .KeyName("MyPK");
         })
    .Properties(pb => 
         {
            pb.Property(e => e.Id)
                .ColumnName("ID");
         });
```

Notice that:
* Everything to do with a key is configured in the Key nested closure 
 * KeyBuilder is now accessible so that is is possible to do additional configuration of the key--for example, the KeyName call above
* Likewise, everything to do with properties is configured in the Properties nested closure 
 * Also, Properties returns the EntityBuilder, so additional configuration for the entity can be added without a new call to Entity()
* Everything can be done with no repetition and in a single block of code

However, there is a strong argument that the code is more difficult in various dimensions, including Intellisense experience, understanding what to call and where, and possibly even reading the code. (Note that there are several ways of formatting/arranging/chaining in the nested closure example. The different ways don't seem to significantly change the experience.)

### First attempt to support both

We decided to try to support both for configuring keys so that we could add methods like KeyName. This resulted in two ways to configure a key. The chaining way that is familiar to EF6:

```
 modelBuilder
    .Entity<Customer>()
    .Key(e => new { e.Id, e.Name });
```

Or the nested closure pattern which allows additional configuration of the key:
```
 modelBuilder
    .Entity<Customer>()
    .Key(k => k.Properties(e => new { e.Id, e.Name })
      .KeyName(“MyPK”));
```

This resulted in code where it is hard to figure out when and how to use each version. For example, IntelliSense now has four similar looking overloads from which to pick:
```
  Key(Action<KeyBuilder> keyBuilder)
  Key(Expression<Func<TEntity, object>> keyExpression)
  Key(Action<KeyBuilder> keyBuilder)
  Key(params string[] propertyNames)
```

### Entity method nested closure

We previously decided on using nested closure at the Entity level. This can be combined with chaining to allow code like this:

```
modelBuilder.Entity<Customer>(b => b
   {
     b.Key(e => new { e.Id, e.Name });
       .KeyName(“MyPK”);
     b.Property(e => e.Id)
       .ColumnName(“ID”);
   });
```
Notice that:
* The Key method returns KeyBuilder
 * Allows chaining of additional key configuration, such as KeyName
 * Consistent with Property method
* The nested closure at Entity level keeps this relatively DRY
* Additional overloads are only for Entity, and the overloads only add parameters
 * In other words, the method call always starts the same way, with the nested closure as an optional additional parameter

However, this breaks copy/paste of Key code from EF6 because Key is now returning the KeyBuilder, not the EntityBuilder. (It would be possible for Key inside the nested closure to have a different return value to Key outside the nested closure, but this would break consistency in the API and require more complex implementation.)

Given that this seems like a nice pattern that meets most of the desirable characteristics for the fluent API we decided to break the copy/paste from EF6.

### Additional nested closure parameters

We could add nested closure parameters to other methods, such as Key and Property. We will consider doing this in cases where it is useful to continue working with the builder (e.g. KeyBuilder/PropertyBuilder) beyond the end of a call chain. For example, this is common for Entity where many properties, keys, etc. often need to be configured, but not common for Key where there is limited configuration done in one go for one key.