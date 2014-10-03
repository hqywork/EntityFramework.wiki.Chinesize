# Provider-specific fluent APIs

In EF7 the core Code First fluent API is designed to be extended by providers through extension methods defined in the provider assemblies. In this meeting we discussed several aspects of making this work well.

## Model concepts

The general idea behind this kind of extensibility is that concepts in the model fall broadly into three categories:
* Concepts that are general enough to first-class in the model and available regardless of the provider being used. For example, MaxLength.
* Concepts that apply to a group of related providers, of which the only group we currently have is relational providers. For example, ColumnType or TableName.
* Concepts that are specific to certain providers. For example, PartitionAndRowKey

Note that the lines between these categories are not clear cut and there has been and will likely continue to be debate about which category is appropriate for various concepts.

## Provider-specific models

We [previously decided](https://github.com/aspnet/EntityFramework/wiki/Entity-Framework-Design-Meeting-Notes---July-17,-2014#provider-specific-conventions) that different providers may need to generate slightly different models either by convention or through explicit APIs. However, most of these differences will be represented by differences in annotations on the same model. While we may need to allow for and handle real differences in models for the current discussion we are focusing mainly on differences in annotations.

For groups of providers we want the ability to specify an annotation that will be used by every provider in the group but which can also be overridden for specific providers. For example, use a certain column type for all relational providers except for SQL CE where a different column type should be used. This could be enforced by structure in the annotation mechanism, but for now we will use a simple naming convention where annotation names are prefixed with an appropriate identifier for the group or provider. For example, Relational:ColumnType, SqlServer:ColumnType, SQLite:ColumnType, etc. Provider-aware code such as Migrations and the update pipeline will understand how to interpret these annotations.

## Extension method clashes

One of the traditional problems with extension methods is discoverability. We generally try to mitigate this by placing extension methods in a namespace for which there is a high chance that a using directive already exists in the consuming class. This means that multiple providers are likely to want to place extension methods in the same namespace. However, this causes a problem when two providers are both referenced in the project and they both have a common name for an extension method--for example, ColumnName for relational and Azure Table Storage. Similarly, conflicts can exist if the same extension method name is needed by a group of providers (e.g. relational) and also specific providers in that group for overriding. In such cases application code could end up looking something like this:

```
AtsExtensions.ColumnName(
    modelBuilder
        .Entity<Customer>()
        .Property(e => e.Name),
    “Foo");
```

One proposal discussed was not caring about such clashes based on the idea that multiple referenced providers will be uncommon. We decided that even if this ends up being the case it is not something we want to rely on, and we are already demoing cases with multiple referenced providers.

The two main proposals discussed were:
* Provider-specific extension method names
* Methods to select a provider or provider group

Provider-specific names would look something like:

```
modelBuilder.Entity<Customer>()
    .Property(e => e.Name)
    .RelationalColumnType("foo");

modelBuilder.Entity<Customer>()
    .Property(e => e.Name)
    .SqlServerColumnType("foo");
```

Methods to select a provider/group would look something like:

```
modelBuilder.Entity<Customer>()
    .Property(e => e.Name)
    .ForRelational()
    .ColumnType("foo");

modelBuilder.Entity<Customer>()
    .Property(e => e.Name)
    .ForRelational()
    .ColumnType("foo");
```

However, using this pattern we can also use nested closures to group several provider-specific calls together:

```
modelBuilder.Entity<Customer>()
    .Property(e => e.Name)
    .ForRelational(p =>
    {
        p.ColumnType("foo");
        p.ColumnName("NAME");
    });
```

The general consensus was that the selection methods were a better experience to discover and use, especially when combined with the nested closure option.

## Concern over degrading the relational experience

A concern with this switch to using extension methods is that it degrades the common relational experience when compared to what we have for EF6. This is because methods that are directly on core items in the old stack will now require a call to ForRelational before they can be used. This is of particular concern for column names which are very commonly configured through the fluent API.

One approach that was discussed is to promote column and table naming to first class concepts in the model--the storage name concept that we had in early versions of EF7. There are two main issues with this:
* What should the methods be called? Column and Table are much better for relational but do not apply well to many other kinds of store. Storage applies to all, but is not a very meaningful name for relational or Azure Table Storage providers. One approach to deal with this is to create an assembly for "tabular" providers that can be used for all providers for which these names make sense, but it is not clear that it is really worth doing this.
* What about schemas? While having some kind of storage name is relatively common, the schema part of a name is much more relational-specific. We could separate schema out to a separate call to solve this or do parsing of a single string, but it is again not clear that it is worth it or that this would be a better experience.

Another approach discussed was to create a relational-specific model builder that can be accessed from OnModelCreating using something like an AsRelational method. The model builder design which is API facades over internal implementations lends itself relatively well to this approach. This approach also applies to any relational method, not just column and table names. We decided to give this a try and hold off on changing/promoting column/table naming for now.

## Extending the core API

The discussions above are focused on the Code First fluent API. However, the core metadata could also be extended in a similar way. We decided to implement these kinds of extensions for both the core metadata and the fluent API, with the fluent API delegating to the core metadata methods.

## The methods

Relational providers will have the following extension methods:
* EntityType:
  * Table (2 overloads, one including schema)
* Property:
  * Column
  * ColumnType
  * DefaultExpression
  * DefaultValue
* Key
  * Name
* ForeignKey
  * Name
* Index
  * Name

SQL Server will have some additional methods:
* Key
  * Clustered
* Index
  * Clustered
* Model
  * UseSequence (3 overloads: empty; name; name, block size, start)
  * UseIdentity
* Property
  * UseSequence (3 overloads)
  * UseIdentity

Azure Table Storage will have:
* EntityType:
  * Table
  * PartitionAndRowKey
* Property:
  * Column
  * Timestamp