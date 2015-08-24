# Error handling in Data Annotations

(Content coming soon...)

# Aligning property discovery with type mapping

(Content coming soon...)

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

#### EF7 Curent
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