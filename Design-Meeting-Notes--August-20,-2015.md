# Error handling in Data Annotations

Data annotations are user-written code so any error in the usage of data annotations should be notified to the user. Since data annotations can be used by frameworks other than EF and may not specify the full configuration of the model, throwing an exception in all erroneous cases would just block the user.  

Therefore whenever data annotations are used in an ambiguous way, a warning will be logged using the logger. Though if the data annotations are specifying an invalid model configuration (esp. while using `ForeignKey` or `InverseProperty`), an error will be logged. Since the logger has not been implemented yet, an exception will be thrown at the places where error should be logged for now.  

Below is the possible errors in data annotations and behavior of EF about them.  
* `Key` 
  - **KeyAttribute in derived type** - If using TPH, this will throw while setting key since key cannot be set on derived entity type.
  - **Composite Primary Key** - Due to column ordering not being implemented, it is not possible to specify composite PK using data annotations. If there are multiple properties with `Key` attribute and if the PK is not specified using fluent API for such entity type then an exception will be thrown.

* `MaxLength`& `StringLength`
  - **Invalid length value** - Since these attributes can be used by other frameworks also, EF will ignore if invalid value is specified. The length value should be greater than zero.

* `Timestamp`
  - **Timestamp on non-byte array property** - In EF6, this threw an exception because it gets mapped to `rowversion` for SqlServer which requires the property to be byte array. In EF7, this will not throw any exception since provider other than SqlServer may provide a store type which is concurrency token and database generated but not byte array.
  
* `ForeignKey` & `InverseProperty`
  - Since these attributes are used together to declare relationships, if the specified configuration is invalid relationship then exception will be thrown. In the case of ambiguity, warning will be logged and relationships will be created based on whatever can be interpreted from the given configuration. Following are some examples:
    - `ForeignKey` on property in both related entity types - 2 relationships will be formed with respective foreignkey property. If both navigations are connected by `InverseProperty` then throw.
    - `ForeignKey` on property and navigation in same entity type don't point to each other - throw.
    - `ForeignKey` on both navigations don't match - 2 relationships will be formed with each navigation and respective foreign key property. If both navigations are connected by `InverseProperty` then throw.
    - Composite FK - Due to no column ordering, composite FK must be specified on navigation using `ForeignKey`. If `ForeignKey` on multiple properties then throw.
    - Invalid list of FK properties on navigation - throw.
    - Navigation pointed by `ForeignKey` on property not found - throw.
    - Navigation pointed by `InverseProperty` not found or self or has different return type - throw.
    - `InverseProperty` on both navigations don't point to each other - throw.

# Aligning property discovery with type mapping

Presently `PropertyDiscoveryConvention` uses fixed set of clr types to identify primitive properties, irrespective of what is supported by the providers. This puts limitation on what clr types can be included in the model as primitive properties. It can also create a model with properties which cannot be mapped by the current provider. Ideally, supported primitive types should be dependent on the current provider. To enforce this, property discovery will be aligned TypeMapper.  

Following will be the workflow:
* `PropertyDiscoveryConvention` will pull all the properties from entity's clr type which can be mapped by the current provider as primitive properties.
* The remaining properties, will be added as navigation properties if possible. This would also discover new entity types if applicable.
* Once the model is fully built, if all the properties in entity's clr type are not added as primitive/navigation properties or marked as ignored by user (using `NotMapped` attribute or fluent API), then exception will be thrown providing information of the property which cannot be mapped by TypeMapper.

# Relationship API naming

We've been thru a few iterations of the relationship API in EF7 to see if we could come up with something that was a bit more intuitive. In past versions we have seen folks find the relationship API be the hardest part of EF to understand.

Based on feedback and our own experience using the API we don't think what we have now is necessarily much better than what was in EF6. We are also making an effort to align APIs with their EF6 equivalent unless there is a compelling reason not to.

There are a couple of changes in the EF7 API which we think are compelling to keep:
* Removing the required/optional aspect of the relationship (i.e HasRequired/HasOptional) and just have it be based on the nullability of the foreign key property(s).
* Removing the principal/dependent aspect of the relationship (i.e. WithRequiredPrincipal/WithRequiredDependent) which was super confusing in EF6 (we can never remember how it works and we designed it).

Based on this, we will be adopting the below API that keep some of the changes but more closely aligns with the EF6 naming. These changes are only API name changes and not funcitonal.

## one to many

#### EF6
``` c#
modelBuilder.Entity<Post>()
    .HasRequired(p => p.Blog)
    .WithMany(b => b.Posts);
```

#### EF7 Curent
``` c#
modelBuilder.Entity<Post>()
    .Reference(p => p.Blog) // Required/optional comes from nullability of FK
    .InverseCollection(b => b.Posts);
```

#### EF7 New
``` c#
modelBuilder.Entity<Post>()
    .HasOne(p => p.Blog) // Required/optional comes from nullability of FK
    .WithMany(b => b.Posts);
```

## one to many with FK

#### EF6
``` c#
modelBuilder.Entity<Post>()
    .HasRequired(p => p.Blog)
    .WithMany(b => b.Posts)
    .HasForeignKey(p => p.BlogId);
```

#### EF7 Current
``` c#
modelBuilder.Entity<Post>()
    .Reference(p => p.Blog) // Required/optional comes from nullability of FK
    .InverseCollection(b => b.Posts)
    .ForeignKey(p => p.BlogId);
```

#### EF7 New
``` c#
modelBuilder.Entity<Post>()
    .HasOne(p => p.Blog) // Required/optional comes from nullability of FK
    .WithMany(b => b.Posts)
    .HasForeignKey(p => p.BlogId);
```

## one to zero-or-one

#### EF6
``` c#
modelBuilder.Entity<ProductInfo>()
    .HasRequired(i => i.Product)
    .WithOptional(p => p.ProductInfo);
```

#### EF7 Current
``` c#
modelBuilder.Entity<ProductInfo>()
    .Reference(i => i.Product)
    .InverseReference(p => p.ProductInfo);
```

``` c#
modelBuilder.Entity<ProductInfo>()
    .Reference(i => i.Product)
    .InverseReference(p => p.ProductInfo)
    .ForeignKey<ProductInfo>(i => i.ProductId);
```

#### EF7 New
``` c#
modelBuilder.Entity<ProductInfo>()
    .HasOne(i => i.Product)
    .WithOne(p => p.ProductInfo);
```

``` c#
modelBuilder.Entity<ProductInfo>()
    .HasOne(i => i.Product)
    .WithOne(p => p.ProductInfo)
    .HasForeignKey<ProductInfo>(i => i.ProductId);
```

We will also rename `PrincipalKey()` to `HasPrincipalKey()` for consistency.


# Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/2899) to provide feedback, ask questions, etc.