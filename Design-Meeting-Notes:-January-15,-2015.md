# ForeignKey API

## Background

Currently, we have two fluent APIs for relationships:
- HasOne/WithMany
  - Navigation based
  - Allows FK specification but does not require it
- ForeignKey
  - Requires FK to be specified, even if it is in shadow state or could be discovered by convention
  - Currently does not support navigations or non-primary principal keys

Currently, we have two model builders:
- Normal convention-based builder
  - Has both relationship APIs
- Basic non-convention builder
  - Less abstract/closer to the underlying metadata model
  - Has just the ForeignKey API

## Open issues

We have four open issues already triaged to do, but they aren't all consistent with each other:
- Remove ForeignKey API from convention model builder ([Issue 782](https://github.com/aspnet/EntityFramework/issues/782))
  - Mainly because two relationship APIs on common path is likely to be confusing
  - In addition, the two APIs cannot really be used together. For example, you can't do a .ForeignKey and then do a .HasMany/WithOne call and have it use the FK unless you specify the FK again, in which case there was no value in specifying it in the first place.
- Basic model builder FK APIs do not support non-PK principal keys ([Issue 756](https://github.com/aspnet/EntityFramework/issues/756))
- Allow navigations to be specified when using the basic model builder APIs ([Issue 781](https://github.com/aspnet/EntityFramework/issues/781))
- Remove BasicModelBuilder ([Issue 908](https://github.com/aspnet/EntityFramework/issues/908))
  - Just use convention model builder without conventions instead.
  - Note that this Will work if everything is specified, otherwise you will get a valid model, but with shadow state stuff put in by convention. (See below)

## Related: No-nav relationships

This is the case where you have a foreign key to define a relationship, but there are no associated navigation properties. We previously decided that this was not common enough to have a fluent API for it (Issue #1152). It would still be possible to do using core metadata APIs. (Note that in EF6 it is not possible at all with Code First and we heard virtually no requests to add it.)

However, it may not be as uncommon as we think. For example, ASP.NET Identity has now moved to a domain model that doesn't have navigations.

If we do decide that we want to include it, then the ForeignKey API could be used for this. However, it is also easy to enable it in the HasOne/WithMany APIs.

## Options

- Remove basic builder and remove ForeignKey from convention builder
  - ForeignKey API is removed, so no close-to-the-metadata builder is available
  - Do not need to implement the missing parts of the ForeignKey API
- Remove basic builder and keep ForeignKey API on convention builder
  - OnModelCreating has two relationship APIs
  - Must design/implement navigation and principal key support
    - We could choose to not implement these, but as mentioned above the two APIs don't work together which would then cause additional confusion.
- Keep basic model builder and remove ForeignKey from convention builder
  - Will still have a close-to-the metadata builder
  - Must design/implement navigation and principal key support

## Decision

We will remove the basic model builder and the ForeignKey API.

Notes:
 - We will add back no-navigation property overloads to the HasOne/WithMany APIs. We don't think this will be very confusing, and will allow code like Identity to use the fluent API to define its models
 - We will consider a mode of using the convention model builder that throws in model validation if there is anything in the model that hasn't been explicitly configured. See [Issue 1414](https://github.com/aspnet/EntityFramework/issues/1414).

# Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/1415) to provider feedback, ask questions, etc.