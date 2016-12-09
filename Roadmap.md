# Entity Framework Core (EF Core)

Below is the schedule and roadmap for EF Core. Please note that these dates and feature plans are all subject to change. As with any project of this size it is difficult to predict exactly when things will land. Even so, we think it's important to be as open and transparent as possible about our plans so that our users can have the right expectations and create their plans accordingly.

## Schedule

The schedule for EF Core is in-sync with the .NET Core and ASP.NET Core schedule [which you can find here](https://github.com/aspnet/Home/wiki/Roadmap). 

## Backlog

Because EF Core is a new code base, the presence of a feature in Entity Framework 6.x does not mean that the feature is implemented in EF Core. For more information, see [Compare EF Core & EF6.x](https://docs.microsoft.com/en-us/ef/efcore-and-ef6/index)

We have provided a list of features that we think are important but are not yet implemented. This is by no means an exhaustive list, but calls out some of the important features that are not yet implemented in EF Core.

### Critical O/RM features

The things we think we need before we say EF Core is the recommended version of EF. Until we implement these features EF Core will be a valid option for many applications, especially on platforms such as UWP and .NET Core where EF6.x does not work, but for many applications the lack of these features will make EF6.x a better option.

* Query
 * **Improved translation** will enable more queries to successfully execute, with more logic being evaluated in the database (rather than in-memory).
  * **GroupBy translation** will move translation of the LINQ GroupBy operator to the database, rather than in-memory.
 * **Lazy loading** enables navigation properties to be automatically populated from the database when they are accessed.
 * **Raw SQL queries for non-Model types** allows a raw SQL query to be used to populate types that are not part of the model (typically for denormalized view-model data).

* Database schema management 
 * **Visual Studio wizard for reverse engineer**, that allows you to visually configure connection, select tables, etc. when creating a model from an existing database.
 * **Update model from database** allows a model that was previously reverse engineered from the database to be refreshed with changes made to the schema.

* Modelling
 * **Complex/value types** are types that do not have a primary key and are used to represent a set of properties on an entity type.
 * **Stored procedure mapping** allows EF to use stored procedures to persist changes to the database (`FromSql` already provides good support for using a stored procedure to query).
 * **View mapping** allows EF to map to database views.
 
### High priority features
### 高优先级功能

There are many features on our backlog and this is by no means an exhaustive list. These features are high priority but we think EF Core would be a compelling release for the vast majority of applications without them
>在我们的积压工作中有很多功能，这里并不是一个详尽的清单。这些功能是高优先级的，但我们认为 EF Core 对于绝大多数应用程序将是一个完整的发布。

* Modelling
* >模型
 * **More flexible property mapping**, such as constructor parameters, get/set methods, property bags, etc.
 * >**更灵活的属性映射**，如构造函数参数，get/set 方法，属性等。
 * **Visualizing a model** to see a graphical representation of the code-based model.
 * >**可视化模型** 查看基于代码模型的图形表示形式。
 * **Simple type conversions** such as string => xml.
 * >**简单类型转换** 如 string => xml。
 * **Spatial data types** such as SQL Server's `geography` & `geometry`.
 * >**空间数据类型** 如 SQL Server 中的 `geography` & `geometry`。
 * **Many-to-many relationships** without join entity. You can already [model a many-to-many relationship with a join entity](https://docs.efproject.net/en/latest/modeling/relationships.html#many-to-many).
 * >**多对多关系** 无需连接实体。你可以阅读 [model a many-to-many relationship with a join entity](https://docs.efproject.net/en/latest/modeling/relationships.html#many-to-many)。
 * **Alternate inheritance mapping patterns** for relational databases, such as table per type (TPT) and table per concrete type TPC.
 * >**继承映射模型** 仅用于关系数据库，如每类型一表（TPT）以及每实现一表（TPC）。

* CRUD
 * **Seed data** allows a set of data to be easily upserted.
 * >**Seed data** 允许一组数据容易的 upserted。
 * **ETag-style concurrency token support** 
 * **Eager loading rules** allow a default set of related data to always be retrieved when an entity is queried.
 * >**Eager loading rules** 当查询实体时，允许一组相关数据默认是被取回。
 * **Filtered loading** allows a subset of related entities to be loaded.
 * >**Filtered loading** 允许相关实体的子集被加载。
 * **Simple command interception** provides an easy way to read/write commands before/after they are sent to the database.
 * >**Simple command interception** 提供了一个简单的方法在命令被发送到数据库之前（后）读（写）此命令。

* Providers
 * **Azure Table Storage**
 * **Redis**
 * **Other non-relational databases**

* Platforms
 * **Universal Windows Platform (UWP)** currently works for local development but there are issues with the .NET Native tool chain that the EF and .NET Native teams are working to address.
 * **Xamarin** works in some scenarios but has not been fully tested as a supported scenario.

## EF Core 1.2

EF Core 1.2 is currently in the planning phase.

## EF Core 1.1 (Released)
## EF Core 1.1（已发布）

The following features are already implemented and included in the 1.1 release.
>以下功能已实现和包含在 1.1 发布中。

* **Explicit Loading** allows you to trigger population of a navigation property on an entity that was previously loaded from the database.
* >**Explicit Loading** 允许你从数据库先前的加载的实体上触发导航属性。
* **DbSet.Find** provides an easy way to fetch an entity based on its primary key value.
* >**DbSet.Find** 提供一个简单的方法根据主键获取实体。
* **Connection resiliency** automatically retries failed database commands. This is especially useful when connection to SQL Azure, where transient failures are common.
* >**Connection resiliency** 自动重试失败的数据库命令。当连接到 SQL Azure 时这是非常有用的，因为瞬时失败是很常用的。
* **EntityEntry APIs from EF6.x** such as `Reload`, `GetModifiedProperties`, `GetDatabaseValues` etc.
* >**EntityEntry APIs from EF6.x** 如 `Reload`, `GetModifiedProperties`, `GetDatabaseValues` 等。
* **Field mapping** allows you to configure a backing field for a property. This can be useful for read-only properties, or data that has Get/Set methods rather than a property.
* >**Field mapping** 允许你为属性配置支持字段。这对于只读属性或拥有 Get/Set 方法而不是属性的数据是有用的。
* **Memory-Optimized Tables** are a feature of SQL Server. You can specify that the table an entity is mapped to is memory-optimized. When using EF Core to create and maintain a database based on your model (either with migrations or `Database.EnsureCreated()`), a memory-optimized table will be created for these entities. 
* >**Memory-Optimized Tables** 是 SQL Server 的一个功能。你可以指定实体映射到内存优化表。当使用 EF Core 创建或维护一个基于模型（通过迁移或 `Database.EnsureCreated()`）的数据库时，将创建这些实体的内存优化表。
* **Simplified service replacement** makes it easier to replace internal services that EF uses.
* >**Simplified service replacement** 使 EF 使用的内部服务替换更容易。

## EF Core 1.0 (Released)
## EF Core 1.0（已发布）

The following features are already implemented and included in the 1.0 release.
>以下功能已实现和包含在 1.0 发布中。

* Modelling
* >模型
 * **Basic modelling** based on POCO entities with get/set properties. The common property types from the BCL are supported (`int`, `string`, etc.).
 * >**Basic modelling** 基于带有 get/set 属性的 POCO 实体。被 BCL 支持的公共属性类型（如 `int`, `string` 等）。
 * **Built-in conventions** that build an initial model based on the shape of the entity classes.
 * >**Built-in conventions** 基于实体类的形状构建一个初始模型。
 * **Fluent API** allows you to override the `OnModelCreating` method on your context to further configure the model that was discovered by convention.
 * >**Fluent API** 允许在你的上下文中重载 `OnModelCreating` 方法来深层次的配置可被约定发现的模型。
 * **Data annotations** are attributes that can be added to your entity classes/properties and will influence the EF model (i.e. adding [Required] will let EF know that a property is required).
 * >**Data annotations** 是可以被添加到你的实体类/属性上的特性，并会影响 EF 模型（例如，添加 [Required] 将让 EF 知道这个属性是必需的）。
 * **TPH inheritance pattern** allows entities in an inheritance hierarchy to be saved to a single table using a discriminator column to identify they entity type for a given record in the database.
 * >**TPH inheritance pattern** 允许在继承层次中的实体保存到单独的表中，使用鉴别列来识别数据库中给定记录的实体类型。
 * **Relationships** between entities based on navigation and foreign key properties.
 * >**Relationships** 基于导航属性和外键属性的实体间关系。
 * **Shadow state properties** (properties that are part of the model but do not have a corresponding property in the CLR class).
 * >**Shadow state properties** （属性是模型的一部分但在 CLR 类中没有对应的属性）。
 * **Alternate keys** and the ability to define a relationship that targets an alternate key.
 * >**Alternate keys** 有能力定义关系的目标是一个候选键。
 * **Model validation** that detects invalid patterns in the model and provides helpful error messages.
 * >**Model validation** 检测模型中的无效模式并提供了有用的错误消息。
 * **Key value generation** including client-side generation and database generation.
 * > **Key value generation** 包含客户端生成和数据库生成。
 * **Relational: Table mapping** allows entities to be mapped to tables/columns.
 * > **Relational: Table mapping** 允许实体被映射到表/列。
		
* Change Tracking
* 改变跟踪
 * **Snapshot change tracking** based on recording the original values of an entity when it is retrieved from the database.
 * >**Snapshot change tracking** 在从数据库取回实体的原始值的基本上记录。
 * **Notification change tracking** allows your entities to notify the change tracker when property values are modified.
 * >**Notification change tracking** 当属性值被修改时，允许你的实体去通知改变跟踪。
 * **Accessing tracked state** of entities (via `DbContext.Entry` and `DbContext.ChangeTracker`).
 * >**Accessing tracked state** 访问实体的访问跟踪状态（通过 `DbContext.Entry` 和 `DbContext.ChangeTracker`）。
 * **Attaching detached entities/graphs**. The new `DbContext.AttachGraph` API helps re-attach entities to a context in order to save new/modified entities.
 * >**Attaching detached entities/graphs**。新的 `DbContext.AttachGraph` API 帮助帮助重新附加实体到上下文，以便保存新的/修改的实体。
		
* SaveChanges
 * **Basic save functionality** allows changes to entity instances to be persisted to the database.
 * >**Basic save functionality** 允许实体实例的变化持久化到数据库。
 * **Optimistic Concurrency** protects against overwriting changes made by another user since data was fetched from the database.
 * >**Optimistic Concurrency** 防止覆盖自数据被从数据库中获取之后另一个用户作出的改变。
 * **Async SaveChanges** can free up the current thread to process other requests while the database processes the commands issued from `SaveChanges`.
 * >**Async SaveChanges** 在数据库处理 `SaveChanges` 发出的命令期间，可以释放当前线程去处理其它的请求。
 * **Transactions** means that `SaveChanges` is always atomic (meaning it either completely succeeds, or no changes are made to the database). There are also transaction related APIs to allow sharing transactions between context instances etc.
 * >**Transactions** 意味着 `SaveChanges` 总是原子性的（代表它要么全部成功，要么不对数据库作出改变）。并且有事务相关的 API，允许在上下文实例间同享事务。
 * **Relational: Batching of statements** provides better performance by batching up multiple INSERT/UPDATE/DELETE commands into a single roundtrip to the database.
 * >**Relational: Batching of statements** 通过在到数据库的单一往返行程中批处理多个 INSERT/UPDATE/DELETE 命令，提供了更好的性能。
		
* Query
* >查询
 * **Basic LINQ support** provides the ability to use LINQ to retrieve data from the database.
 * >**Basic LINQ support** 提供了使用 LINQ 从数据库取回数据的能力。
 * **Mixed client/server evaluation** enables queries to contain logic that cannot be evaluated in the database, and must therefore be evaluated after the data is retrieved into memory.
 * >**Mixed client/server evaluation** 使查询包含在数据库中无法求值的逻辑，因此必须将数据取回到内存之后再求值。
 * **NoTracking** queries enables quicker query execution when the context does not need to monitor for changes to the entity instances (i.e. the results are read-only).
 * >**NoTracking** 当上下文不需要监视实体实体的改变时，可以启用快速执行查询（即结果是只读的）。
 * **Eager loading** provides the `Include` and `ThenInclude` methods to identify related data that should also be fetched when querying.
 * >**Eager loading** 提供 `Include` 和 `ThenInclude` 方法来标识关系数据应该在查询时同时取回。
 * **Async query** can free up the current thread to process other requests while the database processes the query.
 * >**Async query** 在数据库处理查询期间可以释放当前线程来处理其它请求。
 * **Raw SQL queries** provides the `DbSet.FromSql` method to use raw SQL queries to fetch data. These queries can also be composed on using LINQ.
 * >**Raw SQL queries** 提供了 `DbSet.FromSql`  方法使用原始 SQL 查询来取回数据。这些查询同样可以在 LINQ 上组成。

* Database schema management
* >数据架构管理	
 * **Database creation/deletion APIs** are mostly designed for testing where you want to quickly create/delete the database without using migrations.
 * >**Database creation/deletion APIs** 主要被设计来用于测试中，你想快速创建或删除数据而不使用迁移。
 * **Relational database migrations** allow a relational database schema to evolve overtime as your model changes.
 * >**Relational database migrations** 允许你的关系数据库架构随着模型的变化而改变。
 * **Reverse engineer from database** scaffolds an EF model based on an existing relational database schema.
 * >**Reverse engineer from database** 基于现有关系数据库架构反向工程。

* Database providers
* >数据库提供者
 * **EntityFramework.SqlServer** connects to Microsoft SQL Server 2008 onwards.
 * >**EntityFramework.SqlServer** 连接到 Microsoft SQL Server 2008 及以上。
 * **EntityFramework.Sqlite** connects to a SQLite 3 database.
 * >**EntityFramework.Sqlite** 连接到 SQLite 3 数据库。
 * **EntityFramework.InMemory** is designed to easily enable testing without connecting to a real database.
 * >**EntityFramework.InMemory** 为简化测试而不需要连接真实数据库而设计的。
 * **3rd party providers** are available for other database engines. See [Database Providers](https://docs.microsoft.com/en-us/ef/core/providers/) for a complete list.
 * >**3rd party providers** 可用于其它数据库引擎。完整列表请参阅 [Database Providers](https://docs.microsoft.com/en-us/ef/core/providers/)。

* Platforms
 * **Full .NET** includes Console, WPF, WinForms, ASP.NET 4, etc.
 * **.NET Core (including ASP.NET Core)** targeting both Full.NET and .NET Core on Windows, OSX, and Linux.

