# 标记生命周期范围和具体使用场景

### 1. 实例生命周期选项 (Instance Per Lifetime Scope)

**场景：** 在 Web 应用程序中，希望每个 HTTP 请求都使用同一个数据库上下文实例。（在 Web 应用程序中，每个 HTTP 请求都是一个独立的工作单元，它们应该使用独立的数据库上下文实例。如果将数据库上下文注册为单例，那么所有请求都会共享同一个数据库上下文实例，这可能导致数据变更冲突、内存泄漏等问题。）

**示例代码：**

```
// 在 Startup.cs 中配置服务
services.AddScoped<DbContext, MyDbContext>();

// 在控制器中使用
public class MyController : Controller
{
    private readonly DbContext _dbContext;

    public MyController(DbContext dbContext)
    {
        _dbContext = dbContext;
    }

    // 使用 _dbContext 进行数据库操作
}
```



### 2. 单例生命周期 (Single Instance)

**场景：** 在应用程序中需要使用一个全局的日志记录器实例。（如应用程序启动日志、系统异常日志）

**示例代码：**

```
// 注册全局的日志记录器
builder.RegisterType<Logger>().SingleInstance();

// 在需要的地方使用
public class MyClass
{
    private readonly Logger _logger;

    public MyClass(Logger logger)
    {
        _logger = logger;
    }

    public void DoSomething()
    {
        _logger.Log("Doing something...");
    }
}
```



### 3. 每次请求 (Instance Per Dependency) 

**场景：** 在一个 Web API 中，需要为每个请求创建一个新的日志记录器实例。（如请求处理开始、结束时的日志。这些日志记录器通常需要和请求的上下文信息关联，因此应该为每个请求创建一个新的实例）

**示例代码：**

```
// 定义日志记录器接口
public interface ILogger
{
    void Log(string message);
}

// 实现日志记录器类
public class Logger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[LOG] {message}");
    }
}

// 控制器依赖于 ILogger 接口
public class MyController : ControllerBase
{
    private readonly ILogger _logger;

    // 控制器的构造函数会注入一个 ILogger 实例
    public MyController(ILogger logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public ActionResult Get()
    {
        // 使用 _logger 记录日志
        _logger.Log("HTTP GET 请求已接收");
        return Ok();
    }
}
```

在这个示例中，`MyController` 控制器依赖于 `ILogger` 接口。每次 HTTP 请求到达时，ASP.NET Core 框架都会创建一个新的 `MyController` 实例，并将一个新的 `Logger` 实例注入到控制器中，确保每次请求都有自己独立的日志记录器实例。



### 4.匹配生命周期作用域

假设有一个发送电子邮件的组件。系统中的逻辑事务可能需要发送多封电子邮件，因此您可以在逻辑事务的各个部分之间共享该组件。但是，不希望电子邮件组件成为全局单例。



```
var builder = new ContainerBuilder();
builder.RegisterType<EmailSender>()
       .As<IEmailSender>()
       .InstancePerMatchingLifetimeScope("transaction");
```

1. 首先，通过 `InstancePerMatchingLifetimeScope("transaction")` 将 `EmailSender` 注册为每个匹配生命周期范围的实例，并为其指定了一个标签 `"transaction"`。

   

```
builder.RegisterType<OrderProcessor>()
       .As<IOrderProcessor>();
builder.RegisterType<ReceiptManager>()
       .As<IReceiptManager>();

var container = builder.Build();
```

2. 然后，注册了 `OrderProcessor` 和 `ReceiptManager`，它们都需要使用 `IEmailSender` 发送电子邮件通知。



```
using(var transactionScope = container.BeginLifetimeScope("transaction"))
{
  using(var orderScope = transactionScope.BeginLifetimeScope())
  {
    var op = orderScope.Resolve<IOrderProcessor>();
    op.ProcessOrder();
  }

  using(var receiptScope = transactionScope.BeginLifetimeScope())
  {
    var rm = receiptScope.Resolve<IReceiptManager>();
    rm.SendReceipt();
  }
}
```

3. 接下来，创建了一个顶级生命周期范围 `transactionScope`，并使用标签 `"transaction"` 开始了一个事务范围。在事务范围内，还创建了两个子范围 `orderScope` 和 `receiptScope`。

4. 在 `orderScope` 中，解析了 `IOrderProcessor`，它会使用父级事务范围中的 `IEmailSender` 实例来处理订单。

5. 在 `receiptScope` 中，解析了 `IReceiptManager`，同样它会使用父级事务范围中的相同 `IEmailSender` 实例来发送收据。



### 5.共享的依赖 (Owned Instances) 

在共享的依赖 (Owned Instances) 的例子中，临时的 HTTP 请求处理器可以是一个用于处理每个 HTTP 请求的处理器，比如一个请求拦截器、身份验证器或者请求处理管道的一部分。而 `ISharedService` 是一个长期存在的服务，通常代表应用程序的核心服务之一，它可能是应用程序中的一个功能模块，如身份认证服务、缓存服务或者数据访问服务。

**示例代码：**

```
// 定义一个长期存在的身份认证服务接口
public interface IAuthenticationService
{
    bool AuthenticateUser(string username, string password);
}

// 实现长期存在的身份认证服务类
public class AuthenticationService : IAuthenticationService
{
    public bool AuthenticateUser(string username, string password)
    {
        // 实现用户身份认证逻辑
        return true; // 这里简化为直接认证通过
    }
}

// 临时的 HTTP 请求处理器，它依赖于 IAuthenticationService 接口
public class RequestHandler
{
    private readonly Func<Owned<IAuthenticationService>> _authServiceFactory;

    public RequestHandler(Func<Owned<IAuthenticationService>> authServiceFactory)
    {
        _authServiceFactory = authServiceFactory;
    }

    public void HandleRequest(string username, string password)
    {
        using (var authService = _authServiceFactory())
        {
            bool isAuthenticated = authService.Value.AuthenticateUser(username, password);
            if (isAuthenticated)
            {
                Console.WriteLine($"User '{username}' has been authenticated.");
            }
            else
            {
                Console.WriteLine($"Failed to authenticate user '{username}'.");
            }
        }
    }
}
```

在这个例子中，`AuthenticationService` 是一个长期存在的服务，负责用户身份认证。而 `RequestHandler` 则是一个临时的 HTTP 请求处理器，它在每个 HTTP 请求时都会创建一个新的 `AuthenticationService` 实例来进行用户身份认证。