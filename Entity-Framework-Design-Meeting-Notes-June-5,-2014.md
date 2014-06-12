# Entity Framework Design Meeting Notes - June 5, 2014

## Fluent API

We have had a lot of positive feedback on the existing Fluent API in EF 4.1 thru 6.x. There are some areas that we've seen folks find confusing or suggest improvements on. Given that, we want to keep the same general experience but take some improvements where we think it makes sense.

#### DECISION: Remove Has prefix from methods
We've seen a number of folks ask that we remove the unneeded **Has** prefix from our APIs, we agree that this just adds clutter, so the plan is to remove these. For example, here is some EF6 code.

```
modelBuilder.Entity<Blog>()
    .HasKey(b => b.BlogId);

modelBuilder.Entity<Blog>()
    .Property(b => b.Url)
    .HasMaxLength(500);
```

And here is the corresponding code you'll write in EF7.

```
modelBuilder.Entity<Blog>()
    .Has(b => b.BlogId);

modelBuilder.Entity<Blog>()
    .Property(b => b.Url)
    .MaxLength(500);
```

#### DECISION: Provide a nested closure pattern for entity configuration

The nested closure pattern refers to using a code blog that operates on a configuration object instead of chaining methods together. The advantage of this is that you don't have to drill down to the scope of the object you are configuring, therefore preventing further configuration on the top level object you started configuring. For example, in the EF6 API if I call use the **Property** method to select a property to configure then I can't go back and configure another property because the Property method has dropped the scope down to configuring a specific property.

The disadvantage of the nested closure pattern is that it adds complexity, both in terms of the code written and the help provided by IntelliSense. Given that many developers are happy to have to start another line of code to configure a property, we still want to keep the existing method chaining pattern from EF6.

```
modelBuilder.Entity<Blog>()
    .Property(b => b.Url)
    .MaxLength(500)
    .Unicode(false)
    .Required();

modelBuilder.Entity<Blog>()
    .Property(b => b.Name)
    .MaxLength(200);
```

But we will also provide an overload of **Entity** that allows configuration to be done in a nested closure. This is purely a style choice for developers who prefer this syntax. The configuration object you are operation on inside the nested closure will be the exact same configuration object returned from the usual **Entity** method.

```
modelBuilder.Entity<Blog>(c => 
    {
        c.Property(b => b.Url)
         .MaxLength(500)
         .Unicode(false)
         .Required();

        c.Property(b => b.Name)
         .MaxLength(200);         
    });

```
#### DECISION: Ignore methods will remain unchanged

We are going to keep the same model discovery logic that traverses navigation properties to include types in the model. This means you can end up with types in your model that you want to exclude. 

The following code from EF6 will remain unchanged in EF7.

```
modelBuilder.Ignore<BlogInfo>();

modelBuilder.Entity<User>()
    .Ignore(u => u.DisplayName);
```

#### DECISION: Complex type methods not yet needed

In the initial RTM of EF7 we are not planning to enable complex and/or value types. To that end, we don't yet need these methods. We are happy with the general patterns though and when we enable complex and/or value types we will pull forward something similar to the current APIs.

#### DECISION: Provide string based APIs to support shadow state

To support shadow state we will have string-based overloads of all our strongly typed APIs. There are some that may not be strictly necessary, but we'll just enable all of them for the sake of consistency and because it's often hard to anticipate the exact scenarios folks will want to do with weakly-types APIs.

When using the string version of the **Property** API you will need to tell us the type of the property. This can be done with generics (easy when you have a hard coded property) or by providing the **Type** (easier when doing dynamic configuration).

```
// Adds an extra property in shadow state
modelBuilder.Entity<Post>()
    .Property<string>("CustomField")
    .HasMaxLength(500);

// Adds an extra property in shadow state without using generics
modelBuilder.Entity<Post>()
    .Property("CustomField", typeof(string))
    .HasMaxLength(500);

// Define an entity that only exists in shadow state (no CLR type)
modelBuilder.Entity("ExtraBlogInfo")
    .Property<int>("ExtraBlogInfoId");
```

If the string provided to the API matches a type/property that is already part of the model (meaning it may be a CLR type/property) then they will just configure that type/property (i.e. the weakly-typed APIs aren't just for configuring shadow state).

##### DECISION: Entities will use fully qualified CLR name as their name in the model

To avoid the issues we have in EF6 with multiple types in different namespaces, we will just use the fully qualified name of the entity as the name we use in metadata. We will not make any effort to lookup based on the short name. For example, if you have a CLR based entity called **Blogging.Models.Blog** and you call ```Entity("Blog")``` you will be configuring a new shadow state entity rather than the CLR based entity. To configure the CLR based entity you would need to call ```Entity("Blogging.Models.Blog")```.

#### DECISION: Keep using anonymous types for composite keys

The anonymous type syntax that we use in EF6 isn't the most discoverable syntax. It's basically impossible to work out what you are supposed to write if you just look at the API signature. However, we provide good IntelliSense with examples of what to write and we are yet to come up with a better pattern.

```
modelBuilder.Entity<Review>()
    .Key(c => new { c.UserId, c.PostId });
```

For the weakly-typed APIs we will just use a **params** of **string**.

```
modelBuilder.Entity("Blogging.Models.Review")
    .Key("UserId", "PostId");
```

#### DECISION: Property facet APIs

We made the following decisions for methods to specify facets of properties. 

| API | Decision |
|-----|----------|
| .IsMaxLength() | The semantics of this API aren't very clearly defined. It is kind of specifying that something should be un-bounded. We don't think this is needed in EF7 **we won't pull this API forward**. |
| .HasMaxLength(int) | Provide ```MaxLength(int)``` with the same functionality |
| .IsFixedLength() | The idea of something being fixed length isn't really a generic concept and is really specific to the nchar/char datatypes in SQL Server. **We won't pull this API forward** since you can just specify those data types if that is what you want. |
| .IsVariableLength() | As above, **We won't pull this API forward** |
| .IsOptional() | We'll provide this functionality via calling ```Required(false)``` |
| .IsRequired() | Provide ```Required()``` with the same functionality |
| .HasPrecision(7, 3) | This isn't really a generic concept. **We won't pull this API forward** since you can just specify the data types you want in the database. |
| .HasDatabaseGeneratedOption(DatabaseGeneratedOption.None) | No decision as yet - we need a separate meeting about store generated patterns |
| .IsUnicode() and .IsUnicode(bool) | Again, this isn't really a generic concept and is more about specifying the data type in the store. **We won't pull this API forward** since you can just specify the data types you want in the database. |
| .IsConcurrencyToken() and .IsConcurrencyToken(bool) | Pull forward as ConcurrencyToken() and ConcurrencyToken(bool). |
| .IsRowVersion() | Again, this isn't really a generic concept and is more about specifying the data type in the store. **We won't pull this API forward** since you can just specify the data types you want in the database. |
| .HasColumnType(string) | Pull forward as a relational specific ```ColumnType(string)``` method. We'll provide an API to make it easy to specify a particular data type depending on the provider being targeted ```builder.OnSqlServer().Entity<Blog>().Property(b => b.Name).HasColumnType("varchar(max)")```. We may also provide helper methods for specifying types (but we're not committing to it yet) - ```HasColumnType(SqlTypes.Decimal(7, 3))``` |
| .HasColumnName(string) | Pull forward as a relational specific ```ColumnName(string)``` method. There is currently a first class **StorageName** property in the metadata model. We will demote this from the top level API since it may not apply to all data stores. |
| .HasColumnAnnotation(string, object) | This is already replaced with better annotation support in the EF7 metadata model |
| .HasColumnOrder(3) | We'll demote this to a migrations concept as we'll have a deterministic ordering of properties in metadata. The migrations pipeline will have a nice default for the order it puts columns in (i.e. PK, scalars, FK). This is actually better because specifying an order at the moment only affects initial migration/creation since all new columns added to an existing table are added at the end. ```[Column(Order = 123)]``` will only remain meaningful for composite key ordering. |


#### DECISION: Try out a Collection/Reference/ForeignKey based relationship API

We've had lots of feedback that folks find the current relationship API in EF6 to be confusing. One of the reasons is that the following concepts are all merged into the HasXYZ /WithXWY methods:
* Pairing up navigation properties
* Specifying cardinality
* Identifying the principal/dependent end of the relationship 

For EF7 we are going to try out a simplified API that uses the same Collection/Reference names that we have on ChangeTracker API. You can start by specifying either end of the relationship or the FK. Whether the relationship is required or not will be handled by a separate Required method.

Here are some examples of the new syntax.

```
modelBuilder.Entity<Post>() 
    .Reference(p => p.Blog) 
    .Collection(b => b.Posts)
    .Required();
```

```
modelBuilder.Entity<Post>() 
    .Reference(p => p.Blog) 
    .Collection(b => b.Posts) 
    .ForeignKey(p => p.TheBlogId);
```

```
modelBuilder.Entity<Post>()
    .ForeignKey<Blog>(p => p.TheBlogId)
    .Reference(p => p.Blog) 
    .Collection(b => b.Posts);
```

For one-to-one relationships you will need to start with a call to ForeignKey so that we know which end of the relationship is the dependent.

```
modelBuilder.Entity<BlogInfo>()
    .ForeignKey<Blog>(p => p.BlogId)
    .Reference(i => i.Blog) 
    .Reference(b => b.Info);
```

This new API would also allow us to support relationships that don't have both navigation properties. We supported one navigation in EF6, but not having no navigation properties.

```
modelBuilder.Entity<Post>() 
    .Reference(p => p.Blog) 
    .ForeignKey(p => p.TheBlogId);
```

```
modelBuilder.Entity<Post>()
    .ForeignKey<Blog>(p => p.BlogId);
```

In EF7 we also want to support having unique keys on entities that are not the primary key. To specify these as the target of a relationship you would use a Key method.

```
// Assume User.Id is configured as the primary key
modelBuilder.Entity<UserLogin>()
    .ForeignKey<User>(l => l.Username)
    .Reference(l => l.User) 
    .Collection(u => u.Logins)
    .Key(u => u.Username);
```

