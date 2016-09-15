New (and improved) Migrations operations
========================================
When working on [#5345][1], [#6102][2] & [#6405][3], it became apparent that we needed more information on the
`AlterColumnOperation` and that we needed operations to represent changes to annotations on `IEntityType` and `IModel`.
(This was discussed previously in [#3122][4].)

In response, the `AlterColumnOperation` now has all the old property and annotation values for the column. The generated
C# now looks like the following.

```C#
migrationBuilder.AlterColumn<int>(
    name: "Id",
    table: "Person",
    nullable: false,
    oldClrType: typeof(long))
    .OldAnnotation("SqlServer:ValueGenerationStrategy", SqlServerValueGenerationStrategy.IdentityColumn);
```

We've also added `AlterTableOperation` and `AlterDatabaseOperation`.

Providers can tell the model differ which annotations impact the database schema by using
`IMigrationsAnnotationProvider`. The model differ will then take care of comparing the annotations and generating the
appropirate operations. The provider's Migrations SQL generator will then interpret the annotations on the operations to
produce the appropriate DDL.

Annotations are now also copied to the drop operations. A special `AlterTableOperation` with just the old annotations
is generated when migrating to the InitialDatabase migraiton.

Custom operations
-----------------
After reviewing these changes, we talked about the ability for providers to create their own custom operations. While
custom operations would be easier for a provider to work with throughout the stack, they make extending the Migrations
pipeline more difficult. Operations need to be scaffolded into code files, and having a community-written F# code
generator break because the Npgsql provider is generating new operation types would not be good for the ecosystem.

Instead, we decided to push providers down the path of using these generic `AlterDatabase` and `AlterTable` operations.
If there are scenarios that cannot be satisfied by using the annotations approach, we would work with them to extend the
functionality.

Ultimately, all the user typically sees is the generated code, and (similar to metadata) we would like to enable
providers to write (and scaffold) extension methods to make the annotations more friendly. For example, the following
lines of code could be equivelant.

```C#
migrationsBuilder.AlterDatabase()
    .CreatePostgresExtension("PostGIS");
    
migrationsBuilder.AlterDatabase()
    .Annotation("Npgsql:.PostGIS", "'PostGIS', '', ''");
```

If the provider can't generate code for the current language, the seccond, more general syntax could be handled by the
Migrations code generator class.

Provider-specific model differs
-------------------------------
Given the decision to push providers toward using annotaitons instead of custom operations, we don't see any need to
make `IMigrationsModelDiffer` a provider service. This was proposed in PR [#6523][5]. Our hope is that they'll work with
us to extend the functionality of our differ rather than replacing it.

Discussion
==========
Please use [the discussion issue][6] to provide feedback, ask questions, etc.


  [1]: https://github.com/aspnet/EntityFramework/issues/5345
  [2]: https://github.com/aspnet/EntityFramework/issues/6102
  [3]: https://github.com/aspnet/EntityFramework/issues/6405
  [4]: https://github.com/aspnet/EntityFramework/issues/3122
  [5]: https://github.com/aspnet/EntityFramework/pull/6523
  [6]: https://github.com/aspnet/EntityFramework/issues/6547
