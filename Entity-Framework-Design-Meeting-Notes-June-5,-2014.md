# These notes are still in progress, they do not yet include all the discussion/decisions from the design meeting

# Entity Framework Design Meeting Notes - June 5, 2014

## Fluent API

We have had a lot of positive feedback on the existing Fluent API in EF 4.1 thru 6.x. There are some areas that we've seen folks find confusing or suggest improvements on. Given that, we want to keep the same general experience but take some improvements where we think it makes sense.

### General configuration

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

### Relationship configuration