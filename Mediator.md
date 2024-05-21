# Mediator.Net 和中介者模式

Mediator.Net是一个在.NET应用程序中实现中介者模式的库。中介者模式是一种行为设计模式，它允许我们降低多个组件或对象之间的通信复杂性。这种模式通过创建一个中心化的中介者对象来协调不同组件之间的交互，从而减少了组件之间的直接依赖。



## 中介者模式的基本概念

在我们的生活中处处充斥着“中介者”，比如租房、买房、找工作、旅游等等可能都需要那些中介者的帮助。地球上国与国之间的关系异常复杂，会因为各种各样的利益关系来结成盟友或者敌人，国与国之间的关系同样会随着时间、环境因为利益而发生改变，而地球上最大的中介者就是联合国了，它主要用来维护国际和平与安全、解决国际间经济、社会、文化和人道主义性质的问题。软件开发过程也同样如此，对象与对象之间存在着很强、复杂的关联关系，如果没有类似于联合国这样的“机构”会很容易出问题。![img](https://img-blog.csdnimg.cn/20181105012947252.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=,size_16,color_FFFFFF,t_70)

- 中介者模式通过中介者对象来封装一系列的对象交互，将对象间复杂的关系网状结构变成结构简单的以中介者为核心的星形结构，对象间一对多的关联转变为一对一的关联，简化对象间的关系，便于理解；各个对象之间的关系被解耦，每个对象不再和它关联的对象直接发生相互作用，而是通过中介者对象来与关联的对象进行通讯，使得对象可以相对独立地使用，提高了对象的可复用和系统的可扩展性。

- 在中介者模式中，中介者类处于核心地位，它封装了系统中所有对象类之间的关系，除了简化对象间的关系，还可以对对象间的交互进行进一步的控制。



## Mediator.Net

Mediator.Net 是一个用于.NET应用程序的轻量级库，它实现了中介者模式，帮助开发者通过中央调度来简化对象之间的通信，从而降低系统组件间的耦合度。



## 如何使用 Mediator.Net

以下是使用Mediator.Net实现中介者模式的基本步骤：

### 1. 安装包

首先，需要在.NET项目中安装Mediator.Net。可以通过NuGet包管理器来安装：

```bash
Install-Package Mediator.Net
```

### 2. 定义消息和处理器

创建消息类和对应的处理器类。消息是你希望在应用程序中传递的数据对象；处理器是接收消息并执行相应操作的逻辑单元。

```c#
using Mediator.Net.Contracts;

public class SimpleMessage : IRequest
{
    public string Content { get; set; }
}

public class SimpleMessageHandler : IRequestHandler<SimpleMessage, Unit>
{
    public Task<Unit> Handle(IReceiveContext<SimpleMessage> context, CancellationToken cancellationToken)
    {
        Console.WriteLine(context.Message.Content);
        return Task.FromResult(new Unit());
    }
}
```

### 3. 配置中介者

在应用程序中配置中介者，并注册你定义的消息和处理器。

```c#
using Mediator.Net;
using Mediator.Net.Binding;

public class Program
{
    public static void Main(string[] args)
    {
        var mediatorBuilder = new MediatorBuilder();
        var mediator = mediatorBuilder.RegisterHandlers(config =>
        {
            config.AddRequestHandler<SimpleMessage, Unit, SimpleMessageHandler>();
        }).Build();

        mediator.SendAsync(new SimpleMessage { Content = "Hello, Mediator!" });
        Console.ReadKey();
    }
}
```