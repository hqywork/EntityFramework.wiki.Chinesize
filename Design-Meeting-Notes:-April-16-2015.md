# Layout of Generated Code in OnModelCreating()

## Background

In the code Reverse Engineering generated inside the OnModelCreating() method in the generated DbContext class we originally had code laid out like this:

            modelBuilder.Entity<Product>(entity =>
            {
                entity.Property(e => e.SomeProperty)
                    .FacetWithNoForClause1("AAA")
                    .FacetWithNoForClause2("BBB")...
                    .ForRelational()
                        .FacetForRelational1("WWWW")
                        .FacetForRelational2("XXXXX")...
                    .ForSqlServer()
                        .FacetForSqlServer1("YYYYY")
                        .FacetForSqlServer1("ZZZZZ")...;

                entity.Property(e => e.SomeOtherProperty)...
            });

and later changed it to this:

            modelBuilder.Entity<Product>(entity =>
            {
                entity.Property(e => e.SomeProperty).AttributeWithNoForClause1("AAA");

                entity.Property(e => e.SomeProperty).AttributeWithNoForClause2("BBB");

                entity.Property(e => e.SomeProperty).ForRelational().AttributeForRelational1("WWWW");

                entity.Property(e => e.SomeProperty).ForRelational().AttributeForRelational2("XXXXX");

                entity.Property(e => e.SomeProperty).ForSqlServer().AttributeForSqlServer1("YYYYY");

                entity.Property(e => e.SomeProperty).ForSqlServer().AttributeForSqlServer1("ZZZZZ");

                entity.Property(e => e.SomeOtherProperty)...
            });

## Discussion

At the meeting on April 16 2015 we discussed multiple approaches and agreed to do the following.

If there is only one facet of any kind for a given property then either:

```
            modelBuilder.Entity<Product>(entity =>
            {
                entity.Property(e => e.SomeProperty).FacetWithNoForClause1("AAA");

                entity.Property(e => e.SomeOtherProperty)...
            });
```

or

```
            modelBuilder.Entity<Product>(entity =>
            {
                entity.Property(e => e.SomeProperty).ForSqlServer().FacetForSqlServer1("YYYYY");

                entity.Property(e => e.SomeOtherProperty)...
            });
```

If there are multiple non-SqlServer but only one SqlServer facet then this:

```
            modelBuilder.Entity<Product>(entity =>
            {
                entity.Property(e => e.SomeProperty)
                    .FacetWithNoForClause1("AAA")
                    .FacetWithNoForClause2("BBB")
                    .ForSqlServer().FacetForSqlServer1("YYYYY");

                entity.Property(e => e.SomeOtherProperty)...
            });
```

And if there are multiple SqlServer server facets this:
```
            modelBuilder.Entity<Product>(entity =>
            {
                entity.Property(e => e.SomeProperty)
                    .FacetWithNoForClause1("AAA")
                    .FacetWithNoForClause2("BBB")
                    .ForSqlServer(property =>
                    {
                        property.AttributeForRelational1("WWWW");
                        property.AttributeForRelational2("XXXX");
                        property.FacetForSqlServer1("YYYY");
                        property.FacetForSqlServer2("ZZZZ");
                    });

                entity.Property(e => e.SomeOtherProperty)...
            });
```

Note: for all Relational facets we are calling the ForSqlServer() overrides of them instead of the ForRelational() ones. 

Reasons for this approach were:
1) terseness is good - if there is only a small number of facets (the usual case) we'd rather the code took up as few lines as possible,
2) grouping - it's nice to have all the facets affecting one property grouped together and separate from those for another property.

There was also discussion that we are thinking of making the ForRelational() facets top-level i.e. not needing a ForRelational() call first. If/when that happens they would be grouped with the FacetWithNoForClauseN() calls instead.


