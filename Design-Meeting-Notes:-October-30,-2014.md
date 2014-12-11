# Models and stores in Migrations

## Migrations relational model

Migrations in EF7 currently uses a relational specific model and mapping information. This model and mapping is only used by Migrations and used for model diffing, etc. This is different from the rest of the EF7 stack where the core model is used with relational-specific annotations interpreted as needed--for example, query and update pipeline. We decided that Migrations should also do this to avoid building a completely different model and to avoid the associated duplication of logic to manage/interpret this model.

### Operations

Most Migrations operations make use of simple data--i.e. there is no additional model here. The exception is for columns where there is a simple DTO-like ColumnModel due to the way columns can be included in multiple places--for example, both adding a column and creating a table. Some of the newer operations now make use of the relational model mentioned above. We decided to stop doing this and align these new operations with the pattern used for the other operations.

## Declarative operations?

We have made an attempt to be more declarative in EF7 migrations. For example, if an alter column operation requires a constraint to be dropped first and recreated afterwards, then this will not be explicitly scaffolded as would have been the case in EF6, but will instead be worked out by the SQL generator. This makes the Migrations mode store-agnostic (see below) because the actual actions that need to be performed can vary between stores, but this is abstracted so that the same declarative migration can be used in both cases.

While this approach is nice when everything works correctly there are two problems with it. The first is that it is not inherently clear what operations are going to be performed on the store. This can make it harder to figure out what is going on, which is especially important when something goes wrong. The second problem is that once something has gone wrong, then it is harder to fix because it is not possible to easily re-order or change the actual actions taken on the store. For these reasons we decided to go back to being more explicit as we were in EF6. This also makes the SQL generation simpler again.

## Store-agnostic migrations

There are scenarios where the same basic model is used with multiple types of store. In these cases the migrations can sometimes be store-specific--for example, when mapping a column to different store-specific types. There was a lot of discussion about how this should be handled. The main points were:
- Sometimes the model shape may be slightly different between different stores, but its OK if this is not supported, at least initially. The important thing is to support differences in annotations between the different stores. However, it would be beneficial if the approach we take naturally handles different models.
- Migrations that were created for one store cannot be assumed to work for a different store. For example, if 20 migrations are created targeting SQL Server, and then SQLite is added as a new supported store, it is likely that the existing migrations will not work correctly. For this case Migrations should scaffold a migration that creates everything from scratch for SQLite.
- Because we are backing away from very declarative operations (see above) it means that the sequence  of operations for one store may be different from the sequence of operations for another store even when there is no actual store-specific differences in the model. This means that the differ (or some other part of the stack) needs to be able to create different sequences of operations for different stores.

Given these things we made the following tentative decisions:
- Each migration should keep track of which store(s) the migration can be used with.
- The model snapshot needs to keep track of which store(s) were in use when it was created.
- Different stores may generate different sequences. We could try to merge these and generate store-agnostic migrations with conditionals for the store-specific parts. However, it would also be OK, and maybe clearer/easier to generate store-specific migrations all the time and have a separate set of Migrations for each store.

## Migrations code generation

Currently code generation does not always use the fluent API for store-specific annotations. It instead just writes out the annotations. We decided that this is fine for now because it is functionally the same as using the fluent API. We can consider allowing stores to generate fluent API calls in the future if it would make the snapshot much more readable.

# EF7 and MARS

Currently EF7 requires support for multiple active result sets (MARS) in many queries. Ideally EF7 should not require this, but there are many competing concerns in the query pipeline which make this difficult. Certainly for now MARS is required for both sub-queries and some Includes.

When MARS is not enabled the result for a query that needs it is: "InvalidOperationException: There is already an open DataReader associated with this Command which must be closed first."

This is not an issue when using the project templates because MARS is included in the default connection string. However, connections strings for SQL Azure do not come with MARS enabled. Specifically:
- Connection strings copied from portal
- Connection strings in environment variables (default publish experience)

Ultimately we should strive to not require MARS. Also, we need to follow up with SQL Server to get a definitive answer as to why MARS is not enabled in their connection strings and whether the default connections strings can have MARS enabled going forward.

Beyond this, we discussed some mitigations:
- Add MARS to connection strings automatically when we can and when MARS is not explicitly disabled. This would allow more applications to work out-of-the-box when targeting Azure. However, it has the drawback that it masks that MARS is being used. Also, it is in general bad practice to manipulate a connection string that we don't own, especially in a functional way. We decided not to do this.
- MARS info in Database Error Page such that developers get a more helpful message indicating what they need to do. This avoids messing with developers connection string. However, the database error page is only available in dev/test environments and hence this won't help when deploying to production.
- Wrap the exception in the core EF code to provide a more helpful message. We decided to go with this mitigation.

# Automatic DetectChanges

In EF6 we automatically called DetectChanges in all the following places:
- DbSet.Find
- DbSet.Local
- DbSet.Remove
- DbSet.Add
- DbSet.Attach
- DbContext.SaveChanges
- DbContext.GetValidationErrors
- DbContext.Entry
- DbChangeTracker.Entries

However, the chances of getting incorrect behavior if DetectChanges is not called when doing Add, Attach, Remove, or queries is relatively low and the perf impact can be high. Therefore in EF7 we will only call DetectChanges automatically for:
- DbContext.SaveChanges
- DbSet.Local (or equivalent)
- DbContext.Entry
- DbChangeTracker.Entries

These are basically the places where you are about to look at local data and/or entity state and hence detecting changes can be important.

# DbContext/DbSet methods

The current EF7 code base is inconsistent when it comes to which methods are exposed on DbContext and DbSet.

DbContext currently has:
- Add&lt;T&gt;(T)
- Update&lt;T&gt;(T)
- Delete&lt;T&gt;(T)

DbSet&lt;T&gt; currently has:
- Add(T)
- Update(T)
- Remove(T)
- AddRange(IEnumerable&lt;T&gt;)
- RemoveRange(IEnumerable&lt;T&gt;)
- UpdateRange(IEnumerable&lt;T&gt;)

Note that
- DbContext has Delete while DbSet has Remove
- There are currently no methods that can be called dynamically without MakeGenericMethod
- There is no Attach
- We currently have Async versions for some and not for others in an inconsistent way.

Decisions where:
- Go with Remove
- Add Attach for now. This may change as we evolve the graph behaviors/disconnected entities work.
- Add non-generic overloads to the context and remove non-generic DbSet
- Drop the Range methods and investigate a params approach instead
- Add Async versions only for Add since this is currently the only method that needs it
