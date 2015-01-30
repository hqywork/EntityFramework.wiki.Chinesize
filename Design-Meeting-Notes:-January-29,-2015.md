# Key value sentinels

## Background

EF has traditionally allowed a key property to be any value other than null. EF never reasoned about keys values--all values were treated equally.

In EF7 we have started treated the default value for the CLR type (default(T); e.g. zero) differently:
* When value generation is used, values are only generated when the property value is not the default
* When store defaults are configured, the store default is only used when the property value is the default

Also, because we do fixup on Add in places we didn't in EF6 default values add additional complexity to avoid unexpected behaviors. For example, new created entities will have default FK values. Care must then be taken not to try to fix these up to tracked entities with default PK values, for example in the transition from default to a real value in value generation.

## Proposal

The proposal is to require some value to be a sentinel, defaulting to default(T). The sentinel value can be configured to be something other than default(T)--for example -1. However, it may then be necessary to ensure new objects are created with the invalid sentinel set for FK properties.

This has the following advantages:
* Simplifies code because state manager will never be tracking principal with that value
* Means we don’t need to look-up value generation metadata and behave differently base on it
* Allows any FK to have an invalid value. This means that when a relationship has been severed using navigation properties, this change can be tracked in the entity by setting the FK to the invalid sentinel value. This removes the need to track "conceptually null" FK values as we did in the old stack.

The main disadvantages are:
* It may not be possible to use EF with legacy databases where the entire key space is considered valid. However, it should possible to map such columns to nullable CLR types or we could provide an option that would allow EF to be used with such a database at the expense of some of the higher-level behaviors that make use of the sentinel value.
* The CLR default for enums usually needs to be treated as a valid value. The mitigation for this is to configure the sentinel for enum keys to some other value--for example, -1.

Sentinel values must be consistent across an entire key chain--that is any foreign keys must use the same sentinel as their primary key. However, it is probably simpler to just set the sentinel per property and then rely on model conventions and validation to ensure consistency.

## Discussion

Please use the [discussion issue](https://github.com/aspnet/EntityFramework/issues/1509) to provider feedback, ask questions, etc.