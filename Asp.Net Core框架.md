# 生命周期

ASP.NET Core 应用的生命周期由宿主控制。宿主负责应用启动和生命周期管理。生命周期的关键事件包括：

- **启动**：应用开始启动时，宿主构建服务容器，然后启动服务，包括 HTTP 服务器（如 Kestrel）。
- **运行**：应用处于运行状态，处理传入的请求。
- **停止**：应用接收到停止命令时，宿主将触发应用终止前的清理工作。



# 中间件

中间件是 ASP.NET Core 应用处理请求和响应的组件。中间件以管道方式组织，请求通过一系列中间件，每个中间件可以对请求进行处理并传递给下一个中间件，或者直接返回响应。

中间件的设置通常在 `Startup.Configure` 方法中配置，例如使用身份验证、错误处理、静态文件服务等。

```
public void Configure(IApplicationBuilder app)
{
    app.UseStaticFiles();
    app.UseRouting();
    //身份认证中间件
    app.UseAuthentication();
    app.UseAuthorization();
    
    app.UseEndpoints(endpoints => {
        endpoints.MapControllers();
    });
}
```



# 依赖注入

ASP.NET Core 内建了依赖注入（DI）容器，它提供了一个方法来解耦组件和其依赖关系。服务（依赖）在应用启动时注册到 DI 容器中，并在需要时由容器自动提供给请求它们的组件。

服务的注册通常在 `Startup.ConfigureServices` 方法中完成。

```
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddSingleton<IMyService, MyService>();
}
```



# 控制器

控制器是 MVC（模型-视图-控制器）模式中的“C”部分，负责处理请求并返回响应。在 ASP.NET Core 中，控制器类派生自 `Controller` 或 `ControllerBase`，并处理来自路由的请求。

```
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}
```



# 路由

路由是 ASP.NET Core 中将 URL 请求映射到特定控制器和操作的机制。在 `Startup.Configure` 方法中配置路由，可以使用传统路由或属性路由。

```
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
});
```



# 启动过程

启动过程是从运行应用程序的那一刻开始的。主要步骤包括：

- **构建宿主**：通过 `Host.CreateDefaultBuilder` 方法设置并构建宿主。
- **配置服务**：在 `Startup.ConfigureServices` 方法中配置应用需要的所有服务。
- **配置中间件管道**：在 `Startup.Configure` 方法中设置处理 HTTP 请求的中间件管道。

```
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```