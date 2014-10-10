# Meaning of MaxLength

This was a discussion around Issue [Issue #722](https://github.com/aspnet/EntityFramework/issues/722). We decided the following:
* When MaxLength is not specified it does not mean that the data type for that property should be unbounded. It means that no max length has been specified and the provider should choose an appropriate data type based on all applicable considerations. One such consideration is allowing the persistence of a wide range of values without failing.
* The API for MaxLength will be 'Nullable<int>', with null indicating that no max length has been specified. Using null here is consistent with other APIs. The actual storage in metadata may not necessarily be a nullable type.
* Only positive non-null values are valid.
* Using a long (or unsigned int) was not considered worthwhile. In general APIs, such as those in ADO.NET, seem to expect just ints.
* The mapping of the parameterless MaxLength data annotation has not been definitively decided. It could mean:
  * The data type used should be unbounded. This would have to be implemented as a provider-specific extension to change the data type in the store. This is the effective behavior in EF6.
  * A simpler behavior is that it maps directly to not specifying a MaxLength as described above, in which case the data type used may not necessarily be unbounded. It is likely we will go with this behavior.

# Required/nullable concepts

This was a discussion around [Issue #723](https://github.com/aspnet/EntityFramework/issues/723). The decisions made were:
* When a property is marked as nullable/non-nullable that is a statement about the values that may be placed in that property. That is, a property marked as nullable supports null values, while a property marked as non-nullable must never contain null values. It follows from this that marking a property which is of a non-nullable CLR type as allowing nulls will not be allowed. This is different from the EF6 behavior where this is allowed.
* The other possible interpretation of nullable/non-nullable is that it defines the store nullability. We are not using this meaning, although by convention Migrations will still infer the column nullability from the nullability of the property in metadata.
* We will not have a separate annotation for store nullability. For Migrations this can be changed if desired by editing the Migration.
* The update and query pipelines should be able to read from or write to a column even if the store nullability is different from the model nullability, assuming the actual data confirms as described above.

