# 在 ASP.NET Core 中集成 Autofac

ASP.NET Core 改革了传统 ASP.NET 的依赖注入框架，引入了统一的依赖注入解决方案。以下是如何在 ASP.NET Core 项目中集成 Autofac 的步骤。

## 1. 安装必需的 NuGet 包

首先，确保你的项目中安装了适用于 ASP.NET Core 的 Autofac 集成包。

```c#
Install-Package Autofac.Extensions.DependencyInjection
```

## 2. 程序配置

在 `Program.cs` 文件中，设置 Autofac 作为服务提供者工厂。

```c#
// 入口函数：应用程序的执行起点
public static void Main(string[] args)
{
    // 创建并配置一个主机构建器
    var host = Host.CreateDefaultBuilder(args)
        // 使用Autofac作为服务提供者工厂
        .UseServiceProviderFactory(new AutofacServiceProviderFactory())
        // 配置Web主机的默认设置
        .ConfigureWebHostDefaults(webHostBuilder => {
            // 指定启动类，用于应用的启动配置
            webHostBuilder.UseStartup<Startup>();
        })
        // 构建配置后的主机
        .Build();

    // 运行构建的主机，启动应用程序
    host.Run();
}
```

## 3. Startup 配置

在 `Startup.cs` 中，你需要配置服务和 Autofac 容器。

### 配置服务

在 `ConfigureServices` 方法中注册 ASP.NET Core 服务。

```c#
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers(); // 例如添加 MVC 控制器
    }
}
```

### 配置 Autofac 容器

直接向 Autofac 容器注册你的模块和服务。

```c#
public void ConfigureContainer(ContainerBuilder builder)
{
    builder.RegisterModule(new MyApplicationModule()); // 注册自定义模块
}
```

## 4. 控制器即服务

如果你想让 Autofac 管理控制器的生命周期，可以通过以下方式改变默认行为：

```c#
services.AddControllersAsServices(); // 让控制器作为服务添加
```

## 5. 多租户支持

如果你的应用需要支持多租户架构，可以使用 Autofac 的多租户支持。

```c#
// 定义一个方法用于配置多租户容器，该方法接收一个 IContainer 对象作为参数
public static MultitenantContainer ConfigureMultitenantContainer(IContainer container)
{
    // 创建一个租户识别策略实例，该策略决定当前请求属于哪个租户
    var strategy = new MyTenantIdentificationStrategy();
    // 使用传入的容器和租户识别策略创建一个多租户容器
    var mtc = new MultitenantContainer(strategy, container);
    // 为具体的租户（例如租户 'a'）配置容器，注册依赖
    mtc.ConfigureTenant("a", cb => cb.RegisterType<TenantDependency>().As<IDependency>());
    // 返回配置好的多租户容器
    return mtc;
}
```

### 具体应用场景

- 假设有一个大型的在线教育平台，该平台服务于全球多个学校。每个学校虽然使用相同的基本功能，但都有其独特的定制需求，比如不同的用户认证方式、课程管理方式和数据分析工具。在这种情况下，您可以使用多租户架构，为每个学校设置一个“租户”，每个租户都有其配置和数据隔离，确保不同学校之间的数据和配置不会相互干扰。
- 想象有一座拥有多个独立公寓的大楼，每个公寓居住着不同的家庭（即租户）。尽管他们共享同一栋楼的一些基础设施（比如水电管线），但每个家庭都有自己的私人空间和配置，比如不同的家具布置或房间装饰（这对应于每个租户的配置和数据）。Autofac 的多租户支持允许为每个租户（家庭）提供定制的服务（如数据库连接），确保他们的信息安全和独立。

## 总结

Autofac 提供了强大的功能来增强 ASP.NET Core 的依赖注入能力，包括多租户支持和灵活的作用域控制。通过上述步骤，你可以有效地集成 Autofac，提高应用的可维护性和扩展性。