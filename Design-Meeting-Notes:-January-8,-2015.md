# Multi-level Include syntax

Since EF 4.1, multi-level includes have been possible using either string syntax:

```
customers.Include("Contact.HomeAddress");
customers.Include("Orders.OrderDetails")
```

or lambda syntax:

```
customers.Include(c => c.Contact.HomeAddress);
customers.Include(c => c.Orders.Select(o => o.OrderDetails));
```

Note that for the lambda syntax we make use of the LINQ Select method when adding an additional level of include after a collection navigation property. This is because the collection navigation property is an IEnumerable<> of the entity type rather than the entity type itself.

For example, to include all products and their suppliers, plus all reviews and review authors:

```
categories
    .Include(c => c.Products.Select(p => p.Reviews.Select(r => r.Author)))
    .Include(c => c.Products.Select(p => p.Supplier));
```

The Select syntax has two issues:
* It is not very discoverable
* It reuses the LINQ Select operator for something that could be argued is not selecting

The second issue could be addressed with a new method, but the discoverability issue would still remain.

## Proposal

A new proposal is to use a pattern similar to LINQ OrderBy:

```
customers.Include(c => c.Contact).ThenInclude(c => c.Address);
customers.Include(c => c.Orders).ThenInclude(o => o.OrderDetails);
```

This means that:
* Lambdas can be valid dotted property expressions
* It works the same for references and collections

The example from above would now look like this:

```
categories
    .Include(c => c.Products).ThenInclude(p => p.Reviews).ThenInclude(r => r.Author)
    .Include(c => c.Products).ThenInclude(p => p.Supplier);
```

Please use the [design meeting discussion issue](https://github.com/aspnet/EntityFramework/issues/1382) to provide feedback, ask questions, etc.