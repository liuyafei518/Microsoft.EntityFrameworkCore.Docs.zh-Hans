# 生成值

## 值生成模式

有三种值生成模式可用于属性值生成。

### 无生成值

不生成值意味着你将总是提供合法值来保存到数据库。该合法值必须在被添加到上下文之前指派给新实体实例。

### 添加时生成值

添加时生成值意味着为新实体生成值。

基于正在使用的数据库提供程序，值可能在客户端由 EF 生成或者在数据库中生成。如果值由数据库生成，那么 EF 会在你添加实体到上下文时为其指派一个临时值，该值将在 `SaveChanges()` 期间被数据库生成的值替代。

如果你添加一个指派了属性值的实体到上下文中，则 EF 将尝试使用该值，而不是为其生成新的值。没有指派运行时默认值（`string` 类型为 `null`，`int` 类型为 `0`，`Guid` 类型为 `Guid.Empty`）的属性会被认为已经被指派了值。查看 [显式设置生成值属性的值](../5、保存数据/I、显式设置生成值属性的值.md)  以了解更多相关信息。

> 警告
>
> 如何为被添加的实体生成值取决于所使用的数据库提供程序。数据库提供程序可能会自动为一些属性类型设置值生成，但是其他属性类型可能就要你手动设置生成值的方式。
>
> 例如，使用 SQL Server 时会自动生成 `GUID` 属性的值（使用 SQL Server 序列 GUID 算法）。然而，如果你将 `DateTime` 属性指定为添加时生成，那么你必须设置生成值的方法。实现办法之一是配置 `GETDATE()` 的默认值，详见 [默认值](./P、关系数据库建模/I、默认值.md)。

### 添加或更新时生成值

添加或更新时生成意味着每次保存（添加或更新）记录时都生成新的值。

与 _添加时生成值_ 类似，如果你为一个新增的实体实例属性指派了值，该值将被插入到数据库中，而不是插入生成值。更新时也可能显式设置值。查看 [显式设置生成值属性的值](../5、保存数据/I、显式设置生成值属性的值.md)  以了解更多相关信息。

> 警告
>
> 如何为添加或更新的实体生成值取决于所使用的数据库提供程序。数据库提供程序可能会自动为一些属性类型设置值生成，但是其他属性类型可能就要你手动设置生成值的方式。
>
> 例如，使用 SQL Server 时，设置为添加或更新时生成并且具有并发标记的 `byte[]` 属性，会被设置为 `rowversion` 数据类型 - 所以它的值会在数据库中生成。然而，如果你将 `DateTime` 属性指定为添加或更新时生成，那么你必须设置生成值的方法。实现办法之一是配置 `GETDATE()` 的默认值（详见  [默认值](./P、关系数据库建模/I、默认值.md)）来为新的数据行生成值。然后你应该使用数据库触发器来在更新时生成值（就像以下样例触发器一样）。

```SQL
CREATE TRIGGER [dbo].[Blogs_UPDATE] ON [dbo].[Blogs]
    AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    IF ((SELECT TRIGGER_NESTLEVEL()) > 1) RETURN;

    DECLARE @Id INT

    SELECT @Id = INSERTED.BlogId
    FROM INSERTED

    UPDATE dbo.Blogs
    SET LastUpdated = GETDATE()
    WHERE BlogId = @Id
END
```

## 惯例

按照惯例，数据类型为 integer 或 GUID 的主键都将被设置为添加时生成值。其他所有属性都将被设置为无生成值。

## 数据注解

**无生成值（数据注解）**

```C#
public class Blog
{
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int BlogId { get; set; }
    public string Url { get; set; }
}
```

**添加时生成值（数据注解）**

```C#
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public DateTime Inserted { get; set; }
}
```

> 警告
>
> 这仅仅是让 EF 知道要为添加的实体生成值，并不能保证 EF 会设置值的具体生成机制。详见 [添加时生成值](#添加时生成值)。

**添加或更新时生成值（数据注解）**

```C#
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
    [DatabaseGenerated(DataaseGeneratedOption.Computed)]
    public DateTime LastUpdated {get; set; }
}
```

> 警告
>
> 这仅仅是让 EF 知道要为添加或更新的实体生成值，并不能保证 EF 会设置值的具体生成机制。详见 [添加或更新时生成值](#添加或更新时生成值)。

## 流式 API

可以使用流式 API 来更改给定属性的值生成模式。

**无生成值（流式 API）**

```C#
modelBuilder.Entity<Blog>()
    .Property(b => b.BlogId)
    .ValueGeneratedNever();
```

**添加时生成值（流式 API）**

```C#
modelBuilder.Entity<Blog>()
    .Property(b => b.Inserted)
    .ValueGeneratedOnAdd();
```

> 警告
>
> `ValueGeneratedOnAdd()` 仅仅是让 EF 知道要为添加的实体生成值，并不能保证 EF 会设置值的具体生成机制。详见 [添加时生成值](#添加时生成值)。

**添加或更新时生成值（流式 API）**

```C#
modelBuilder.Entity<Blog>()
    .Property(b => b.LastUpdated)
    .ValueGeneratedOnAddOrUpdate();
```

> 警告
>
> 这仅仅是让 EF 知道要为添加或更新的实体生成值，并不能保证 EF 会设置值的具体生成机制。详见 [添加或更新时生成值](#添加或更新时生成值)。