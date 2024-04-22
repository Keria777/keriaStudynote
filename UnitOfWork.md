# Mediator.Net UnitOfWork 中间件使用指南

## 概述

UnitOfWork（工作单元）是一个重要的企业级设计模式，用于处理数据访问层的业务事务。在`Mediator.Net`框架中，`UnitOfWork`中间件提供了一种机制，通过它可以在处理程序中保持一组操作在单个事务中完成，确保这些操作要么全部成功，要么全部回滚。保睁了数据的一致性和完整性，尤其是在处理并发数据操作时。

## 设置UnitOfWork中间件

### 安装

首先，确保已经安装了`Mediator.Net`及其`UnitOfWork`中间件：

```c#
Install-Package Mediator.Net
Install-Package Mediator.Net.Middlewares.UnitOfWork
```

### 配置

使用`MediatorBuilder`创建并配置`Mediator`，并将`UnitOfWork`中间件添加到全局接收管道中：

```c#
var builder = new MediatorBuilder();

builder.RegisterHandlers(typeof(ExceptionInHandlerShouldRollbackTransaction).Assembly)
    .ConfigureGlobalReceivePipe(x =>
    {
        x.UseUnitOfWork(() => true); // 启用UnitOfWork中间件
    });
```

## 使用UnitOfWork

### 场景描述

假设有一个场景，需要在一个操作中添加车辆(`Car`)和人员(`Person`)到数据库。这两个操作需要在一个事务中完成，以保证数据的一致性。如果任一操作失败，则整个事务应该回滚。

### 处理程序实现

下面是两个事件处理程序的示例，它们共享同一个`CommittableTransaction`事务对象，确保操作在单个工作单元中执行。

#### 添加车辆

```c#
class WhenACarIsAdded : IEventHandler<PersonAndCarAddedEvent>
{
    private readonly MyDbContext _db;

    public WhenACarIsAdded(MyDbContext db)
    {
        _db = db;
    }

    public async Task Handle(IReceiveContext<PersonAndCarAddedEvent> context)
    {
        CommittableTransaction tx;
        if (context.TryGetService(out tx))
        {
            if (_db.Database.Connection.State != ConnectionState.Open)
            {
                _db.Database.Connection.Open();
            }
            _db.Database.Connection.EnlistTransaction(tx);
        }

        var car = new Car {Id = context.Message.CarId, Name = context.Message.CarName};
        _db.Cars.Add(car);
        await _db.SaveChangesAsync();

        if (context.Message.ShouldThrow)
        {
            throw new Exception(nameof(WhenACarIsAdded));
        }
    }
}
```

#### 添加人员

```c#
class WhenAPersonIsAdded : IEventHandler<PersonAndCarAddedEvent>
{
    private readonly MyDbContext _db;

    public WhenAPersonIsAdded(MyDbContext db)
    {
        _db = db;
    }

    public async Task Handle(IReceiveContext<PersonAndCarAddedEvent> context)
    {
        CommittableTransaction tx;
        if (context.TryGetService(out tx))
        {
            if (_db.Database.Connection.State != ConnectionState.Open)
            {
                _db.Database.Connection.Open();
            }
            _db.Database.Connection.EnlistTransaction(tx);
        }

        var per = new Person { Id = context.Message.PersonId, FirstName = context.Message.FirstName };
        _db.Persons.Add(per);
        await _db.SaveChangesAsync();
    }
}
```

> [!IMPORTANT]
>
> **`Handle`方法**：是事件处理的核心，它异步执行以响应事件。首先尝试从上下文中获取事务（`CommittableTransaction`），如果获取成功，并且数据库连接未打开，则打开连接并将其注册到事务中。然后，创建一个新的`Car`对象，将其添加到上下文中，并调用`SaveChangesAsync`来异步保存更改到数据库。

## UnitOfWork的好处和使用场景

使用工作单元模式的主要好处包括：

- **一致性保障**：确保多个操作要么全部成功，要么全部失败。
- **简化错误处理**：集中事务管理简化了错误处理代码。
- **提高性能**：通过减少数据库连接的开闭操作，提升应用性能。

适用场景包括：

- **复杂业务逻辑**：涉及多步骤或多资源的业务逻辑处理。
- **数据一致性要求高的操作**：如金融交易、库存管理等关键操作。
- **批量数据处理**：在批处理数据导入或更新时使用。

## 结论

`Mediator.Net`的`UnitOfWork`中间件为.NET开发者提供了一个强大的工具，用以管理复杂的业务逻辑中的数据一致性和事务完整性。正确地使用此模式可以显著提高应用的稳定性和可靠性。


# 实战

```c#
using Mediator.Net;
using Mediator.Net.Context;
using Mediator.Net.Contracts;
using Mediator.Net.Pipeline;
using PractiseForKeria.Core.Data;

namespace PractiseForKeria.Core.MiddleWares.UnitOfWork
{
    public static class UnitOfWorkMiddleWare
    {
        public static void UseUnitOfWork<TContext>(this IPipeConfigurator<TContext> configurator,
            IUnitOfWork unitOfWork = null) where TContext : IContext<IMessage>
        {
            if (unitOfWork == null && configurator.DependencyScope == null)
            {
                throw new DependencyScopeNotConfiguredException(
                    $"{nameof(unitOfWork)} is not provided and IDependencyScope is not configured," +
                    $" Please ensure {nameof(unitOfWork)} is registered properly if you are using IoC container," +
                    $" otherwise please pass {nameof(unitOfWork)} as parameter");
            }

            unitOfWork ??= configurator.DependencyScope.Resolve<IUnitOfWork>();

            configurator.AddPipeSpecification(new UnitOfWorkSpecification<TContext>(unitOfWork));
        }
    }
}
```

- **`UseUnitOfWork`方法**：这是一个泛型扩展方法，用于`IPipeConfigurator<TContext>`。它允许开发者将`UnitOfWork`功能添加到任何消息处理管道。
- **依赖注入**：方法检查`unitOfWork`是否已经提供，或者是否可以从依赖注入容器中解析。如果两者都不可用，则抛出`DependencyScopeNotConfiguredException`异常。
- **注册中间件**：如果`unitOfWork`有效，则将其封装在一个`UnitOfWorkSpecification<TContext>`中，并添加到配置器中。

## 中间件规格定义



```c#
using System.Runtime.ExceptionServices;
using Mediator.Net.Context;
using Mediator.Net.Contracts;
using Mediator.Net.Pipeline;
using PractiseForKeria.Core.Data;

namespace PractiseForKeria.Core.MiddleWares.UnitOfWork;

public class UnitOfWorkSpecification<TContext> : IPipeSpecification<TContext> where TContext : IContext<IMessage>
{
    private readonly IUnitOfWork _unitOfWork;

    public UnitOfWorkSpecification(IUnitOfWork unitOfWork)
    {
        _unitOfWork = unitOfWork;
    }

    public bool ShouldExecute(TContext context, CancellationToken cancellationToken)
    {
        return true;
    }

    public Task BeforeExecute(TContext context, CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }

    public Task Execute(TContext context, CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }

    public async Task AfterExecute(TContext context, CancellationToken cancellationToken)
    {
        if (_unitOfWork.ShouldSaveChanges)
        {
            await _unitOfWork.SaveChangesAsync(cancellationToken).ConfigureAwait(false);
            _unitOfWork.ShouldSaveChanges = false;
        }
    }

    public Task OnException(Exception ex, TContext context)
    {
        ExceptionDispatchInfo.Capture(ex).Throw();
        return Task.CompletedTask;
    }
}
```

- **`UnitOfWorkSpecification<TContext>`**：这是实现`IPipeSpecification<TContext>`接口的类，负责定义中间件的行为。
- **`AfterExecute`方法**：这是中间件的核心，只有在执行后才调用。如果标记为需要保存更改(`ShouldSaveChanges`为`true`)，则会异步保存更改到数据库，并重置标记。

## 注册中间件



```c#
private void RegisterMediator(ContainerBuilder builder)
{
    var mediatorBuilder = new MediatorBuilder()
        .RegisterHandlers(_assemblies)
        .ConfigureGlobalReceivePipe(x =>
    {
        x.UseUnitOfWork();//使用UnitOfWork
    });
    builder.RegisterMediator(mediatorBuilder);
}
```

- **配置**：这段代码在全局接收管道中注册了`UnitOfWork`中间件，这样所有通过中介器的消息都会经过这个中间件。
- **依赖注入**：使用`ContainerBuilder`来注册中介器和它的配置，确保所有依赖项都正确解析。