# 数据注释（Data Annotations）

### `[Key]`

标识实体的主键。

```c#
public class User
{
    [Key]
    public int Id { get; set; }
}
```

### `[Required]`

指示字段是必需的。

```c#
public class User
{
    [Required]
    public string Name { get; set; }
}
```

### `[StringLength]`

指定字符串属性的最大长度和最小长度。

```c#
public class User
{
    [StringLength(100, MinimumLength = 3)]
    public string Name { get; set; }
}
```

### `[Range]`

指定数值或日期范围。

```c#
public class Product
{
    [Range(1, 100)]
    public int Quantity { get; set; }
}
```

### `[EmailAddress]`

验证电子邮件地址格式。

```c#
public class User
{
    [EmailAddress]
    public string Email { get; set; }
}
```

### `[RegularExpression]`

使用正则表达式进行模式匹配。

```c#
public class User
{
    [RegularExpression(@"^[a-zA-Z''-'\s]{1,40}$")]
    public string FirstName { get; set; }
}
```

## 实体框架（Entity Framework）

### `[Table]`

指定数据库中的表名称。

```c#
[Table("Users")]
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

### `[Column]`

指定数据库中的列名称。

```c#
public class User
{
    [Column("UserName")]
    public string Name { get; set; }
}
```

### `[ForeignKey]`

指定外键关系。

```c#
public class Order
{
    public int UserId { get; set; }

    [ForeignKey("UserId")]
    public User User { get; set; }
}
```

### `[InverseProperty]`

指定导航属性的反向关系。

```c#
public class User
{
    public int Id { get; set; }

    [InverseProperty("User")]
    public ICollection<Order> Orders { get; set; }
}
```

### `[NotMapped]`

指示属性不应映射到数据库表中的列。

```c#
public class User
{
    [NotMapped]
    public string FullName => $"{FirstName} {LastName}";  // 计算属性，不需要存储在数据库中
}
```

## 序列化和反序列化

### `[JsonIgnore]`

指示在序列化和反序列化过程中忽略该属性。

```c#
public class User
{
    [JsonIgnore]
    public string Password { get; set; }
}
```

### `[JsonProperty]`

指定属性在 JSON 中的名称。

```c#
public class User
{
    [JsonProperty("user_name")]
    public string Name { get; set; }
}
```

### `[XmlRoot]`

用于指定 XML 文档的根元素名称。

```c#
[XmlRoot("Person")]
public class Person
{
    public string Name { get; set; }
}
```

### `[XmlElement]`

用于指定属性或字段在 XML 中的元素名称。

```c#
public class Person
{
    [XmlElement("FullName")]
    public string Name { get; set; }
}
```

### `[XmlAttribute]`

用于指定属性或字段在 XML 中的属性名称。

```c#
public class Person
{
    [XmlAttribute("Age")]
    public int Age { get; set; }
}
```

## 数据验证

### `[Compare]`

用于比较两个属性的值。

```c#
public class RegisterModel
{
    public string Password { get; set; }

    [Compare("Password")]
    public string ConfirmPassword { get; set; }
}
```

### `[CreditCard]`

验证属性是否是有效的信用卡号。

```c#
public class Payment
{
    [CreditCard]
    public string CreditCardNumber { get; set; }
}
```

### `[Phone]`

验证属性是否是有效的电话号码。

```c#
public class Contact
{
    [Phone]
    public string PhoneNumber { get; set; }
}
```

### `[Url]`

验证属性是否是有效的 URL。

```c#
public class Website
{
    [Url]
    public string Homepage { get; set; }
}
```

## 安全和授权

### `[AllowAnonymous]`

用于允许未授权用户访问特定的控制器或操作方法。

```c#
[AllowAnonymous]
public IActionResult PublicEndpoint()
{
    return View();
}
```

### `[Authorize(Roles = "Admin")]`

用于限制访问仅限于特定角色的用户。

```c#
[Authorize(Roles = "Admin")]
public IActionResult AdminEndpoint()
{
    return View();
}
```

## MVC 和 Web API

### `[Bind]`

用于指定模型绑定时应包含或排除的属性。

```c#
public class User
{
    [BindRequired]
    public string Name { get; set; }

    [BindNever]
    public string Password { get; set; }
}
```

### `[Produces]`

指定控制器或操作方法的响应类型。

```c#
[Produces("application/json")]
public IActionResult GetJsonData()
{
    return Json(new { Name = "John", Age = 30 });
}
```

### `[Consumes]`

指定控制器或操作方法的请求类型。

```c#
[Consumes("application/json")]
public IActionResult PostJsonData([FromBody] User user)
{
    // 处理代码
}
```

### `[ApiController]`

标记一个类作为 API 控制器，并启用一些默认行为，如自动模型验证。

```c#
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetUser(int id)
    {
        // 获取用户逻辑
    }
}
```

## 方法和类标记

### `[DebuggerStepThrough]`

告诉调试器在调试时不要单步执行通过标记的方法或属性。

```c#
[DebuggerStepThrough]
public void MyMethod()
{
    // 代码
}
```

### `[AsyncStateMachine]`

标记编译器生成的状态机类型，用于异步方法。

```c#
[AsyncStateMachine(typeof(MyAsyncStateMachine))]
public Task MyAsyncMethod()
{
    // 代码
}
```

### `[CallerMemberName]`

用于自动捕获调用成员名称的编译器服务属性，通常用于日志记录。

```c#
public void Log(string message, [CallerMemberName] string memberName = "")
{
    Console.WriteLine($"{memberName}: {message}");
}
```

### `[Obsolete]`

标记一个方法或类为过时的，并提示使用者。

```c#
[Obsolete("Use NewMethod instead")]
public void OldMethod()
{
    // 代码
}
```

## 调试和性能

### `[DebuggerDisplay]`

控制调试时对象的显示方式。

```c#
[DebuggerDisplay("Name = {Name}, Age = {Age}")]
public class User
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

### `[DebuggerBrowsable]`

控制调试器中成员的显示方式。

```c#
public class User
{
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    public int Age { get; set; }
}
```

### `[MethodImpl]`

控制方法的编译方式。

```c#
[MethodImpl(MethodImplOptions.Synchronized)]
public void MyMethod()
{
    // 代码
}
```

## 其他常用标记

### `[Flags]`

指定一个枚举类型可以作为位域处理。

```c#
[Flags]
public enum FileAccess
{
    Read = 1,
    Write = 2,
    Execute = 4
}
```

### `[Serializable]`

指定一个类可以序列化。

```c#
[Serializable]
public class User
{
    public string Name { get; set; }
}
```

### `[NonSerialized]`

指定一个字段在序列化过程中应被忽略。

```c#
[Serializable]
public class User
{
    public string Name { get; set; }

    [NonSerialized]
    private int age;
}
```

### `[ThreadStatic]`

指定一个字段为每个线程独有的静态字段。

```c#
public class MyClass
{
    [ThreadStatic]
    private static int _field;
}
```

### `[Conditional]`

指定方法仅在特定条件下调用。

```c#
public class MyClass
{
    [Conditional("DEBUG")]
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
```

