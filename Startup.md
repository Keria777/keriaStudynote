```c#
public void ConfigureServices(IServiceCollection services)
{
    // 配置服务
}
```

**ConfigureServices 方法**

- `ConfigureServices` 方法是用来配置应用程序的服务的。在这个方法中，你可以注册应用程序所需的服务，比如数据库上下文、身份验证服务、MVC 中间件等。



```c#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapGet("/", async context =>
        {
            await context.Response.WriteAsync("Hello, world!");
        });
    });
}
```

**Configure 方法**

- `Configure` 方法用于配置应用程序的请求处理管道。在这个方法中，你可以定义中间件的顺序和逻辑，以处理传入的 HTTP 请求。
- `env.IsDevelopment()` 检查当前应用程序是否在开发环境下运行。如果是开发环境，就启用开发者异常页面，方便调试。
- `app.UseRouting()` 启用路由中间件，用于处理传入的 HTTP 请求并将它们路由到适当的处理程序。
- `app.UseEndpoints()` 配置终结点路由，用于指定 URL 路由和相应的处理程序。在这个示例中，将根路径 `/` 映射到一个异步的请求处理程序，该处理程序简单地返回一个 "Hello, world!" 字符串。





# 考核项目startup

### 命名空间和依赖

```c#
using Autofac;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using PractiseForZero.Core;
using PractiseForZero.Core.Extensions;
using PractiseForZero.Core.Services;
```

- **Autofac**: 一个依赖注入容器库，用于管理应用中的依赖关系。
- **JwtBearer**: 用于配置JWT（Json Web Token）承载身份验证的命名空间。
- **Microsoft.IdentityModel.Tokens**: 包含创建和验证安全令牌（如JWT）所需的类。
- **PractiseForZero.Core**: 项目自定义核心库，可能包含领域模型和逻辑。
- **Extensions**: 自定义的扩展方法集。
- **Services**: 定义服务接口或类，这些是业务逻辑的实现。

### 类定义

```c#
namespace PractiseForZero.Api;

public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    private IConfiguration Configuration { get; }
```

- 定义 `Startup` 类，负责配置应用的启动过程。
- 通过构造函数接收一个 `IConfiguration` 对象，该对象包含了应用的配置信息，如appsettings.json的内容。
- `IConfiguration` 实例被存储在私有属性中供整个类使用。

### ConfigureServices 方法

```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();
    services.AddHttpContextAccessor();
```

- `AddControllersWithViews()`: 添加MVC控制器和视图的支持。
- `AddHttpContextAccessor()`: 注册一个服务，以便可以在应用中任何地方通过依赖注入获取 `HttpContext`。

#### 配置 JWT 认证

```c#
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options =>
        {
            options.TokenValidationParameters = new TokenValidationParameters()
            {
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new RsaSecurityKey(Configuration.GetValue<string>("JWT:PublicKey").CreateRsaFromPemString()),
                ValidateIssuer = false,
                ValidateAudience = false,
                ValidateLifetime = true,
                ClockSkew = TimeSpan.Zero
            };
            options.Events = new JwtBearerEvents
            {
                OnTokenValidated = async context =>
                {
                    var blacklistService = context.HttpContext.RequestServices.GetRequiredService<ITokenBlacklistService>();
                    var token = context.Request.Headers["Authorization"].ToString();
                    if (token != null && blacklistService.IsBlacklisted(token.Substring(7)))
                    {
                        context.Fail("This token is blacklisted");
                    }
                }
            };
        });
```

- `AddAuthentication` 和 `AddJwtBearer` 方法配置 JWT 身份验证。
- `TokenValidationParameters` 设置用于验证传入令牌的参数。
- 事件处理程序 `OnTokenValidated` 在每个令牌验证后被调用，用于检查令牌是否在黑名单中。

### ConfigureContainer 方法

```c#
public void ConfigureContainer(ContainerBuilder builder)
{
    builder.RegisterModule(new PractiseForZeroModule(Configuration, typeof(PractiseForZeroModule).Assembly));
}
```

- 配置 Autofac 容器，注册一个自定义模块，可能包含服务和组件的依赖注入配置。

### Configure 方法

```c#
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }
    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapDefaultControllerRoute();
    });
}
```

- `Configure` 方法设置应用的 HTTP 请求管道。
- 根据环境使用不同的错误处理页面。
- 启用 HTTPS 重定向、静态文件服务、路由、认证和授权。
- 定义默认的路由模板。

这个 `Startup` 类为 ASP.NET Core 应用