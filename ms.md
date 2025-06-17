## 🔹 一、C# 语言基础

### 1️⃣ 值类型和引用类型有什么区别？

**参考答案：**

- 值类型存储在**栈**上，赋值时是**值的拷贝**，如：`int`, `bool`, `struct`。
- 引用类型存储在**堆**上，赋值时是**引用的拷贝**，如：`class`, `string`, `array`。
- 值类型不允许为 null，除非使用 `Nullable<T>`；引用类型可以为 null。

------

### 2️⃣ 关键字 `ref` 和 `out` 有什么区别？

**参考答案：**

- `ref` 需要在方法外部**先初始化**变量；`out` 不需要。
- `ref` 是读写参数；`out` 是只写参数（方法内必须赋值）。

```
void TestRef(ref int a) => a += 1;
void TestOut(out int a) { a = 10; }
```

------

## 🔹 二、.NET 基础与ASP.NET Core

### 3️⃣ ASP.NET Core 中中间件（Middleware）是什么？怎么写一个自定义中间件？

**参考答案：**

- 中间件是一个组件，用于处理 HTTP 请求管道中的逻辑。
- 你可以拦截请求，做身份验证、日志记录、异常处理等。

```
public class MyMiddleware
{
    private readonly RequestDelegate _next;
    public MyMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine("Before request");
        await _next(context);
        Console.WriteLine("After request");
    }
}
```

- 注册方式：`app.UseMiddleware<MyMiddleware>();`

------

### 4️⃣ 什么是依赖注入（DI）？ASP.NET Core 是如何实现的？

**参考答案：**

- 依赖注入是一种设计模式，通过构造函数、属性或方法将依赖传递进来，而不是在类内部创建依赖。
- ASP.NET Core 内置了一个轻量级的 DI 容器，通过 `IServiceCollection` 实现。

```
services.AddTransient<IMyService, MyService>();
```

------

## 🔹 三、Entity Framework Core

### 5️⃣ EF Core 中 `DbContext` 是做什么的？生命周期该如何选择？

**参考答案：**

- `DbContext` 是数据库的会话对象，封装了数据库操作。
- 生命周期建议：
  - Web 应用：`InstancePerLifetimeScope`（每次请求创建一次）
  - 后台服务：`Singleton` 要小心线程安全问题



### 6️⃣ ASP.NET Core 的三种内置生命周期

| 生命周期类型 | 说明                                   | 适用场景                           | 注意事项                           |
| ------------ | -------------------------------------- | ---------------------------------- | ---------------------------------- |
| `Singleton`  | 全局唯一，应用启动时创建，直到程序结束 | 配置类、工具类、缓存等             | 必须线程安全，避免注入 Scoped 服务 |
| `Scoped`     | 每个请求中唯一共享                     | Web 应用中的业务服务、DbContext    | 最常用，生命周期与请求绑定         |
| `Transient`  | 每次注入都新建一个实例                 | 无状态服务，如工具类、临时处理逻辑 | 使用频繁，但避免资源浪费           |

Singleton 适合redis

Transient 适合Job和TokenClaims

Scoped大部份都用



### 6️⃣ 你如何实现数据库迁移和版本管理？

**参考答案：**

- 使用命令：
  - `Add-Migration MigrationName`
  - `Update-Database`
- 可以使用 `Fluent API` 或 Data Annotations 控制表结构

------

## 🔹 四、异步编程与线程

### 7️⃣ async 和 await 是怎么工作的？

**参考答案：**

- `async` 方法会被编译成状态机；
- `await` 会将执行权还给调用方，当前线程不会阻塞；
- 最终会生成一个 `Task` 对象。

`async` 和 `await` 是 C# 中的异步编程模型，它们底层是基于 Task 的状态机。通过 `await`，我们可以在不阻塞线程的情况下“等待”一个任务的完成，使得代码逻辑像同步一样清晰，尤其在 Web 应用中可以大幅提高并发能力。我在项目中经常使用 async/await 来处理数据库访问、Http 请求等场景。

------

### 8️⃣ Task 和 Thread 有什么区别？

**参考答案：**

- `Thread` 是一个系统级线程，开销大。
- `Task` 是基于线程池的抽象，效率更高，适合异步操作。

------

## 🔹 五、设计模式

### 9️⃣ 简述你了解的设计模式，举一个你用过的。

**参考答案：**

- 单例模式（Singleton）、工厂模式（Factory）、中介者模式（Mediator）等。
- 我用过 **依赖注入（DI）模式**，配合接口和 `AddScoped` 实现控制反转。

## 🔹 六、CI/CD

### 🔟 谈谈你理解的CI/CD

```
🧑‍💻 开发提交代码 (push)
   ⬇️
🔁 CI 系统触发：
   - 自动拉取代码
   - 编译 / 构建
   - 执行单元测试
   - 静态代码检查（如 SonarQube）
   - 构建 Docker 镜像
   - 上传构建产物（artifact）
   ⬇️
🚀 CD 系统：
   - 自动部署到测试环境
   - 手动 or 自动发布到生产环境
```

我理解的 CI/CD 是一种通过自动化手段来实现代码从提交、测试、构建到部署的完整流程。CI 主要解决集成频繁、测试难的问题，CD 则负责把构建产物稳定、安全地交付到目标环境。它可以提高开发效率、减少人工操作失误。我在项目中曾使用 TeamCity 搭建过 .NET Core 的自动化构建和部署流程，结合 Docker 和脚本部署到内网服务器。



## 🔹 七、Mediator.Net

#### 🔟 谈谈你对 `Mediator.Net` 的理解？你为什么选它而不用 MediatR？

**参考答案：**

`Mediator.Net` 是一个基于中介者模式的库，它支持更加**模块化和扩展性的管道（Pipeline）机制**，适合大型系统解耦模块间通信。相比于 MediatR，它有以下优势：

1. 提供更丰富的管道控制（前置/后置处理器）
2. 支持与 RabbitMQ、Kafka 等集成，实现消息驱动架构
3. 更适合企业级场景下的**命令-事件解耦处理**

在项目中，我通过它实现了命令处理、领域事件发布，并在不同模块之间解耦。例如，STO Out 提交后自动触发出库事件，通过 `Mediator.Net` 分发到多个 Handler 进行日志记录、库存扣减等后续处理。

## 🔹 八、MassTransit+RabbitMQ

#### 在 `Barcode` 项目中，我们通过 MassTransit + RabbitMQ 实现了异步解耦的模块间通信，例如：

- 出库成功后，系统自动发送一个 `StockOutEvent`
- 财务系统收到这个事件后进行会计凭证生成（通过 IConsumer 消费）

### ✅ 1. 错误队列（_error）

失败消息会被自动发送到 `queue_name_error`，可以人工回查和处理。

### ✅ 2. 支持消息中间件中间件（Filter）

可在管道中插入日志、安全校验、跟踪 ID 等

### ✅ 3. 延迟消息（调度器）

支持设置延时投递（基于 RabbitMQ 延迟插件）

```
await bus.Publish<IOrderTimeout>(new {...}, ctx =>
{
    ctx.Delay = TimeSpan.FromMinutes(5);
});
```

### ✅ 4. 分布式事务支持（Outbox）

通过 Outbox 保证数据和消息一起提交，避免消息丢失。

### **✅ 5. 完整推送流程**

其他系统上有StaffUSUpdatedEvent，通过`IBus.Publish()`方法（项目中是`context.Publish()`多）推送到RabbitMQ的Exchange上，MassTransit 会自动完成消息的序列化、Exchange 声明、路由和投递。

```
HR 系统
   ↓
调用 IBus.Publish(new StaffUSUpdatedEvent { ... })
   ↓
MassTransit 底层：
- 序列化事件为 JSON
- 自动创建 Exchange（如果不存在）
- 将消息推送到 RabbitMQ 的 Exchange 中
   ↓
RabbitMQ Exchange（Fanout 类型）
   ↓
推送到所有绑定的 Queue（如你系统的 staff_update_queue）
```

MassTransit 会自动监听队列，接收到消息后反序列化并调用 `StaffUSUpdatedEventConsumer()` 方法同步更新barcode系统中的数据

#### ✅ 对比表格：`context.PublishAsync()` vs `bus.Publish()`

| 特性                             | `context.PublishAsync()`                        | `bus.Publish()`                |
| -------------------------------- | ----------------------------------------------- | ------------------------------ |
| 来源                             | `IReceiveContext<T>`（Mediator.Net 内部上下文） | `IBus`（MassTransit 核心总线） |
| 是否感知当前请求上下文           | ✅ 感知，绑定当前 Command 的处理链               | ❌ 不感知上下文                 |
| 是否推荐在 CommandHandler 中使用 | ✅ 推荐，生命周期/上下文一致                     | ✅ 可用，但不推荐               |
| TraceId / Headers 会保留         | ✅ 一般会带上下文 Header、CorrelationId          | ❌ 需要手动设置                 |
| 适用场景                         | 当前 Command 执行后发布领域事件                 | 全局事件广播、调度后台任务     |
| 测试友好性                       | ✅ 更好（框架内模拟完整流程）                    | ✅                              |





# HiFood

### 🔹 九、AsParallel并行机制

#### ✅ 使用 `AsParallel()` 的典型场景：

- 数据量大（几万条以上）
- 每个元素的处理逻辑比较复杂或耗时（如计算、IO）
- 处理逻辑没有共享状态（无副作用，线程安全）

#### ❌ 不适合：

- 数据量小
- 需要处理顺序（顺序敏感）
- 涉及线程不安全的资源操作（如写文件、写共享集合）



### 🔹 十、冬夏令时

#### ❓Q1：你说你处理了时区同步问题，具体遇到了什么问题？为什么要处理 PST 和 PDT 的切换？

> 在我们项目中，定时同步任务从多个系统拉取数据（如 SAP、HiFood），对接方系统使用的是 **美国太平洋时间（PST/PDT）**。
>
> 如果我们用 UTC 或服务器本地时间直接处理，就会在**夏令时切换时**（3月和11月）出现以下问题：
>
> - 同步数据重复或漏同步（多同步1小时或跳过1小时）
> - 跨天同步边界出错（认为今天是昨天或明天）
>
> 所以我设计了一个基于 **IANA 时区 ID（America/Los_Angeles）** 的时间转换逻辑，动态适配**冬令时 PST（UTC-8）和夏令时 PDT（UTC-7）**的切换。



#### ❓Q2：你是怎么实现自动识别和切换夏令时的？

> 我使用 .NET 中的 `TimeZoneInfo` 类，系统已内置夏令时规则（基于 Windows 或 IANA）。具体做法是：

```
var tz = TimeZoneInfo.FindSystemTimeZoneById("America/Los_Angeles");
var localTime = TimeZoneInfo.ConvertTimeFromUtc(DateTime.UtcNow, tz);
```

> 这个方法会根据当前日期自动判断是否处于夏令时（PDT）或冬令时（PST），不需要我们手动判断。



### 🔹 十一、任务补偿

#### ❓Q1：你说的是“任务补偿机制”，具体指的是什么？为什么你需要它？

> 在我们项目中，有一个定时后台任务通过 Hangfire 调度，串联调用多个系统接口（如 HiFood、内部权限系统、SAP）提交数据。其中任何一个接口失败，可能导致部分数据写入成功、部分失败，造成数据不一致。
>
> 由于这不是数据库事务场景，而是**多个异步接口组成的业务链路**，我们采用“任务补偿机制”来保障最终一致性：
>
> - 如果某一步失败，我们会记录状态并重试该步骤
> - 如果多次失败，我们将失败记录入补偿队列（数据库）
> - 后续会有专门的补偿 Job 自动再次调用失败接口，直到成功或报警处理



#### ❓Q2：那你是怎么保证多接口调用的数据一致性的？比如 SAP 接口最后才调，那前面失败了怎么办？

✅  接口串联逻辑中，用数据库事务+状态标识控制

```
using var tx = await _unitOfWork.BeginTransactionAsync();

try {
    await _interfaceA.Call();
    await _interfaceB.Call();
    await _interfaceC.Call(); // SAP
    await _unitOfWork.CommitAsync();
} catch {
    await _unitOfWork.RollbackAsync();
    throw;
}
```

#### ❓Q3：你怎么处理补偿 Job 的调度与错误告警？

> - Hangfire 的 Retry 策略最多尝试 3 次（`[AutomaticRetry(Attempts = 3)]`），超过次数失败会进入 `_failed` 队列
> - 我写了一个补偿 Job（每日定时跑），扫描失败任务表（比如 `SyncJobLog.Status = Failed && RetryCount < Max`），再次执行
> - 如果失败次数达到上限，写入报警系统（钉钉 / 邮件），人工介入
> - 每次补偿执行前先判断是否已成功，确保不会多次重复操作





