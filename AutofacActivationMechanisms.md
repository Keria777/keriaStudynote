# Autofac 自动激活组件

Autofac 容器提供了几种机制，允许在容器构建时自动激活组件。这些功能可以用于初始化设置，预热和确保应用程序的组件在启动时就绪。

## 1. 可启动组件（IStartable）

`IStartable` 接口允许组件在容器最初构建时自动启动。这类似于开门前预热咖啡机。

### 实现方式

- 实现 `Autofac.IStartable` 接口。
- 在接口的 `Start()` 方法中定义启动逻辑。
- 注册组件时，将其标记为 `IStartable` 并通常设置为单例 (`SingleInstance()`).

### 示例代码

```csharp
public class StartupMessageWriter : IStartable
{
    public void Start()
    {
        Console.WriteLine("App is starting up!");
    }
}

var builder = new ContainerBuilder();
builder.RegisterType<StartupMessageWriter>().As<IStartable>().SingleInstance();
var container = builder.Build();
```

## 2. 自动激活组件

自动激活的组件在容器构建完毕时自动实例化，类似于店里的自动门或感应灯。

### 注册方法

使用 `AutoActivate()` 方法在构建容器时自动激活组件。

### 示例代码

```c#
var builder = new ContainerBuilder();
builder.RegisterType<TypeRequiringWarmStart>().AsSelf().AutoActivate();
var container = builder.Build();
```

## 3. 构建回调

构建回调允许在容器构建完成后执行自定义逻辑，类似于开门前的最后检查。

### 实现方式

注册一个 `Action<IContainer>`，在容器构建的最后阶段执行。

### 示例代码

```c#
var builder = new ContainerBuilder();
builder.RegisterBuildCallback(c => c.Resolve<DbContext>());
var container = builder.Build();
```

## 注意事项

- **避免过度使用启动逻辑**：应用程序启动逻辑应简洁明了，避免复杂化。
- **生命周期管理**：正确管理组件的生命周期和作用域，避免在不适当的时候重新激活或创建组件。

## 总结

使用 Autofac 的自动激活机制可以高效地管理和准备应用程序组件，确保它们在需要时即刻就绪，从而优化应用程序的启动和运行效率。