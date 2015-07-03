# SQL Server value generation scenarios

The idea here is to go over common scenarios where real values are generated and see whether the experience makes sense. Cases where everything works are consider first, followed by cases where exceptions are thrown.

## Positive cases

### Insert with Identity column (default experience)

```C#
using (var context = new BlogContext())
{
    context.AddRange(new Blog { Name = "One Unicorn" }, new Blog { Name = "Two Unicorns" });

    context.SaveChanges();
}

using (var context = new BlogContext())
{
    var blogs = context.Blogs.OrderBy(e => e.Id).ToList();

    Assert.Equal(1, blogs[0].Id);
    Assert.Equal(2, blogs[1].Id);
}
```

### Insert using sequence-based HiLo pattern

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.UseSqlServerSequenceHiLo();
}
```
```C#
using (var context = new BlogContext())
{
    context.AddRange(new Blog { Name = "One Unicorn" }, new Blog { Name = "Two Unicorns" });

    context.SaveChanges();
}

using (var context = new BlogContext())
{
    var blogs = context.Blogs.OrderBy(e => e.Id).ToList();

    Assert.Equal(1, blogs[0].Id);
    Assert.Equal(11, blogs[1].Id);
}
```

Notes:
- Set the default pool size to 1
- Allow the pool size to be configured in UseSqlServerSequenceHiLo for scaling on web apps

### Use sequence as column default

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Sequence("MySequence")
        .IncrementBy(1);

    modelBuilder
        .Entity<Blog>()
        .Property(e => e.Id)
        .DefaultValueSql("next value for MySequence");
}
```
```C#
using (var context = new BlogContext())
{
    context.AddRange(new Blog { Name = "One Unicorn" }, new Blog { Name = "Two Unicorns" });

    context.SaveChanges();
}

using (var context = new BlogContext())
{
    var blogs = context.Blogs.OrderBy(e => e.Id).ToList();

    Assert.Equal(1, blogs[0].Id);
    Assert.Equal(2, blogs[1].Id);
}
```

Notes
- When creating a sequence explicitly (i.e. not for HiLo) set the increment size to 1 by default
- Property can be marked as read-only before save and the behavior is the same, but an exception will be thrown if a value is set explicitly

## Do not use key generation

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Blog>()
        .Property(e => e.Id)
        .StoreGeneratedPattern(StoreGeneratedPattern.None);
}
```
```C#
using (var context = new BlogContext())
 {
     context.AddRange(new Blog { Id = 66, Name = "One Unicorn" }, new Blog { Id = 67, Name = "Two Unicorns" });

     context.SaveChanges();
 }

 using (var context = new BlogContext())
 {
     var blogs = context.Blogs.OrderBy(e => e.Id).ToList();

     Assert.Equal(66, blogs[0].Id);
     Assert.Equal(67, blogs[1].Id);
 }
 ```

Notes
- Use methods instead of enum to set the store generated pattern:
  - ValueGeneratedNever()
  - ValueGeneratedOnAdd()
  - ValueGeneratedOnAddOrUpdate()

### No key generation changing sentinel

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Blog>()
        .Property(e => e.Id)
        .StoreGeneratedPattern(StoreGeneratedPattern.None)
        .Metadata.SentinelValue = -1;
}
```
```C#
using (var context = new BlogContext())
{
    context.AddRange(new Blog { Id = 0, Name = "One Unicorn" }, new Blog { Id = 1, Name = "Two Unicorns" });

    context.SaveChanges();
}

using (var context = new BlogContext())
{
    var blogs = context.Blogs.OrderBy(e => e.Id).ToList();

    Assert.Equal(0, blogs[0].Id);
    Assert.Equal(1, blogs[1].Id);
}
```

Notes
- Allow drop down to Metadata to use nested closure pattern
- Consider ensuring temp value generator does not try to use sentinel value

### No key generation using nullable key to allow null sentinel

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Blog>()
        .Property(e => e.Id)
        .StoreGeneratedPattern(StoreGeneratedPattern.None);
}
```
```C#
using (var context = new BlogContext())
{
    context.AddRange(
        new NullableKeyBlog { Id = 0, Name = "One Unicorn" },
        new NullableKeyBlog { Id = 1, Name = "Two Unicorns" });

    context.SaveChanges();
}

using (var context = new BlogContext())
{
    var blogs = context.NullableKeyBlogs.OrderBy(e => e.Id).ToList();

    Assert.Equal(0, blogs[0].Id);
    Assert.Equal(1, blogs[1].Id);
}
```

Notes
- Make this work by making the CLR type the source for sentinel selection

### Column default SQL in non-key column

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(e => e.CreatedOn)
        .StoreGeneratedPattern(StoreGeneratedPattern.Identity)
        .DefaultValueSql("getdate()");
}
```
```C#
using (var context = new BlogContext())
{
    var blogs = new List<Blog>
    {
        new Blog { Name = "One Unicorn" },
        new Blog { Name = "Two Unicorns", CreatedOn = new DateTime(1969, 8, 3, 0, 10, 0) }
    };

    context.AddRange(blogs);

    context.SaveChanges();

    Assert.NotEqual(new DateTime(), blogs[0].CreatedOn);
    Assert.NotEqual(new DateTime(), blogs[1].CreatedOn);
}

using (var context = new BlogContext())
{
    var blogs = context.Blogs.OrderBy(e => e.Id).ToList();

    Assert.Equal(new DateTime(1969, 8, 3, 0, 10, 0), blogs[0].CreatedOn);
    Assert.NotEqual(new DateTime(), blogs[1].CreatedOn);

    blogs[0].Name = "One Pegasus";
    blogs[1].CreatedOn = new DateTime(1973, 9, 3, 0, 10, 0);

    context.SaveChanges();
}

using (var context = new BlogContext())
{
    var blogs = context.Blogs.OrderBy(e => e.Id).ToList();

    Assert.Equal(new DateTime(1969, 8, 3, 0, 10, 0), blogs[0].CreatedOn);
    Assert.Equal(new DateTime(1973, 9, 3, 0, 10, 0), blogs[1].CreatedOn);
}
```

Notes
- Setting DefaultValueSql will automatically set ValueGeneratedOnAdd
- Property can be marked as read-only before save and the behavior is the same, but an exception will be thrown if a value is set explicitly

### Computed column

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<FullNameBlog>()
        .Property(e => e.FullName)
        .StoreGeneratedPattern(StoreGeneratedPattern.Computed)
        .SqlServerComputedExpression("FirstName + ' ' + LastName");
}
```
```C#
using (var context = new BlogContext())
{
    var blog = context.Add(new FullNameBlog { FirstName = "One", LastName = "Unicorn" }).Entity;

    context.SaveChanges();

    Assert.Equal("One Unicorn", blog.FullName);
}

using (var context = new BlogContext())
{
    var blog = context.FullNameBlogs.Single();

    Assert.Equal("One Unicorn", blog.FullName);

    blog.LastName = "Pegasus";

    context.SaveChanges();

    Assert.Equal("One Pegasus", blog.FullName);
}
```

Notes
- Make SqlServerComputedExpression relational instead of SQL Server specific
- Change name to ComputedColumnSql
- Make it set ValueGeneratedOnAddAndUpdate automatically
- Consider allowing a SQL fragment that can be inserted into statement sent by the update pipeline for the column

### Client-side GUID keys (default behavior)

```C#
Guid afterSave;

using (var context = new BlogContext())
{
    var blog = context.Add(new GuidBlog { Name = "One Unicorn" }).Entity;

    var beforeSave = blog.Id;

    context.SaveChanges();

    afterSave = blog.Id;

    Assert.Equal(beforeSave, afterSave);
}

using (var context = new BlogContext())
{
    Assert.Equal(afterSave, context.GuidBlogs.Single().Id);
}
```

Notes
- Generates SQL Server sequential GUIDs on the client

### Server-side GUID generation on insert

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<GuidBlog>()
        .Property(e => e.Id)
        .DefaultValueSql("newsequentialid()");
}
```
```C#
Guid afterSave;

using (var context = new BlogContext())
{
    var blog = context.Add(new GuidBlog { Name = "One Unicorn" }).Entity;

    var beforeSave = blog.Id;

    context.SaveChanges();

    afterSave = blog.Id;

    Assert.NotEqual(beforeSave, afterSave);
}

using (var context = new BlogContext())
{
    Assert.Equal(afterSave, context.GuidBlogs.Single().Id);
}
```

Notes
- Setting DefaultValueSql to anything will cause temporary GUIDs to be generated on the client

## Negative cases

### Specify keys while using Identity column

```C#
using (var context = new BlogContext())
{
    context.AddRange(new Blog { Id = 1, Name = "One Unicorn" }, new Blog { Id = 2, Name = "Two Unicorns" });

    // DbUpdateException : An error occurred while updating the entries. See the
    // inner exception for details.
    // SqlException : Cannot insert explicit value for identity column in table 
    // 'Blog' when IDENTITY_INSERT is set to OFF.
    Assert.Throws<DbUpdateException>(() => context.SaveChanges());
}
```

Notes
- Try to educate people to either set keys or use Identity column, but not both
- For seeding scenarios, other patterns like the sequence ones above allow mixing, but care must be taken with key space
- Same exception is ultimately thrown if sentinel value is used when first non-sentinel value is encountered
- Consider trying to provide more information that will help the developer decide what they should do

### No key generation with attempt to use sentinel

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder
        .Entity<Blog>()
        .Property(e => e.Id)
        .StoreGeneratedPattern(StoreGeneratedPattern.None);
}
```
```C#
using (var context = new BlogContext())
{
    context.AddRange(new Blog { Id = 0, Name = "One Unicorn" }, new Blog { Id = 1, Name = "Two Unicorns" });

    // The property 'Id' on entity type 'Blog' has a temporary value while attempting to change
    // the entity's state to 'Unchanged'. Either set a permanent value explicitly or ensure
    // that the database is configured to generate values for this property.
    Assert.Equal(
        Internal.Strings.TempValuePersists("Id", "Blog", "Unchanged"),
        Assert.Throws<InvalidOperationException>(() => context.SaveChanges()).Message);
}
```

Notes
- Make the exception be more helpful as to what is actually going on (insert with sentinel) and what can be done (don't use sentinel, change sentinel, or use nullable CLR types)

### Insert value when read-only before save is set

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Sequence("MySequence");

    modelBuilder
        .Entity<Blog>()
        .Property(e => e.Id)
        .DefaultValueSql("next value for MySequence")
        .Metadata.IsReadOnlyBeforeSave = true;
}
```
```C#
using (var context = new BlogContext())
{
    context.AddRange(new Blog { Id = 1, Name = "One Unicorn" }, new Blog { Name = "Two Unicorns" });

    // The property 'Id' on entity type 'Blog' is defined to be read-only before it is 
    // saved, but its value has been set to something other than a temporary or default value.
    Assert.Equal(
        Internal.Strings.PropertyReadOnlyBeforeSave("Id", "Blog"),
        Assert.Throws<InvalidOperationException>(() => context.SaveChanges()).Message);
}
```

Notes
- Consider using different term for "sentinel"

### Insert value into computed column

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<FullNameBlog>()
        .Property(e => e.FullName)
        .StoreGeneratedPattern(StoreGeneratedPattern.Computed)
        .SqlServerComputedExpression("FirstName + ' ' + LastName");
}
```
```C#
using (var context = new BlogContext())
{
    context.Add(new FullNameBlog { FirstName = "One", LastName = "Unicorn", FullName = "Gerald" });

    // The property 'FullName' on entity type 'FullNameBlog' is defined to be read-only before it is 
    // saved, but its value has been set to something other than a temporary or default value.
    Assert.Equal(
        Internal.Strings.PropertyReadOnlyBeforeSave("FullName", "FullNameBlog"),
        Assert.Throws<InvalidOperationException>(() => context.SaveChanges()).Message);
}
```

### Update value in computed column

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<FullNameBlog>()
        .Property(e => e.FullName)
        .StoreGeneratedPattern(StoreGeneratedPattern.Computed)
        .SqlServerComputedExpression("FirstName + ' ' + LastName");
}
```
```C#
using (var context = new BlogContext())
{
    context.Add(new FullNameBlog { FirstName = "One", LastName = "Unicorn" });

    context.SaveChanges();
}

using (var context = new BlogContext())
{
    var blog = context.FullNameBlogs.Single();

    blog.FullName = "The Gorilla";

    // The property 'FullName' on entity type 'FullNameBlog' is defined to be read-only after it has been saved, 
    // but its value has been modified or marked as modified.
    Assert.Equal(
        Internal.Strings.PropertyReadOnlyAfterSave("FullName", "FullNameBlog"),
        Assert.Throws<InvalidOperationException>(() => context.SaveChanges()).Message);
}
```

### Handle optimistic concurrency exception

```C#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<ConcurrentBlog>()
        .Property(e => e.Timestamp)
        .StoreGeneratedPattern(StoreGeneratedPattern.Computed)
        .ConcurrencyToken();
}
```
```C#
using (var context = new BlogContext())
{
    var blog = context.Add(new ConcurrentBlog { Name = "One Unicorn" }).Entity;

    context.SaveChanges();

    using (var innerContext = new BlogContext())
    {
        var updatedBlog = innerContext.ConcurrentBlogs.Single();
        updatedBlog.Name = "One Pegasus";
        innerContext.SaveChanges();
        var currentTimestamp = updatedBlog.Timestamp.ToArray();

        try
        {
            blog.Name = "One Earth Pony";
            context.SaveChanges();
        }
        catch (DbUpdateConcurrencyException)
        {
            // Update original values (and optionally any current values)
            // Would normally do this with just one method call
            context.Entry(blog).Property(e => e.Id).OriginalValue = updatedBlog.Id;
            context.Entry(blog).Property(e => e.Name).OriginalValue = updatedBlog.Name;
            context.Entry(blog).Property(e => e.Timestamp).OriginalValue = updatedBlog.Timestamp;

            // Calling SaveChanges will throw because "Timestamp is read-only" (Original and current values don't match)
            //context.SaveChanges();

            // Try to fix this by marking as not modified
            context.Entry(blog).Property(e => e.Timestamp).IsModified = false;

            // Still throws because DetectChanges marks as modified again
            // context.SaveChanges();

            // Try to fix this by making sure current and original values are same and marking as not modified
            context.Entry(blog).Property(e => e.Timestamp).CurrentValue = updatedBlog.Timestamp;
            context.Entry(blog).Property(e => e.Timestamp).IsModified = false;

            // Finally saves!
            context.SaveChanges();

            Assert.NotEqual(blog.Timestamp, currentTimestamp);
        }
    }
}
```

Notes
- This is clearly not usable as it stands
- Add a new flag to metadata which means don't ever try to send this value no matter whether it has been modified or not
- This is essentially what the old stack did with computed columns
- Set this by default for computed concurrency tokens
- Consider making it so that setting IsModified to false will reject changes to the property
