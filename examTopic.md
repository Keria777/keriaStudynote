

# 

# 项目架构笔记

## 概述
本文档旨在提供关于项目中三个主要模块：Core模块、API模块和Message模块的详细描述和理解。每个模块的职责将被清晰地界定，以支持项目的可维护性和扩展性。

## API模块

### 职责

- **请求处理**：接收外部的请求并返回响应。
- **身份验证和授权**：确保API请求的安全性。

### 组件

- **Controllers**：处理进入的HTTP请求，调用Core模块的服务。

```c#
[ApiController]
[Route("[controller]")]
public class ProductsController : ControllerBase {
    private readonly IProductService _productService;

    public ProductsController(IProductService productService) {
        _productService = productService;
    }
    
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id) {
        var product = _productService.GetProductById(id);
        if (product == null) {
            return NotFound();
        }
        return Ok(product);
    }

}
```



## Core模块

### 职责
- **业务逻辑处理**：实现应用程序的核心业务逻辑。
- **数据持久化**：管理所有数据库交互，包括数据的CRUD操作。
- **领域模型定义**：定义业务领域内的实体和它们的行为。

### 组件
- **Domain**：包含所有领域实体和领域服务。
- **Services**：业务逻辑的实现地，服务层包含应用程序的主要功能。
- **Data**：数据访问层，包括数据库模型和仓库实现。
- **DbUp**：负责数据库迁移和版本控制。
- **Handler**：处理业务逻辑的执行，如命令和查询的处理器。
- **Mapping**：负责数据转换和对象映射的设置，通常用于Domain对象与DTO或视图模型的转换。
- **Settings**：应用程序的配置设置，可能包括数据库连接字符串等。
- **EnumS**：定义在整个应用程序中使用的枚举类型。

### 示例代码
```csharp
public class ProductService : IProductService {
    private readonly IRepository<Product> _productRepository;

    public ProductService(IRepository<Product> productRepository) {
        _productRepository = productRepository;
    }

    public Product GetProductById(int id) {
        return _productRepository.FindById(id);
    }
}
```



## Message模块

### 职责

- **数据传输**：定义和使用数据传输对象（DTOs）来传递数据。
- **消息传递**：处理和传递消息队列中的消息。

### 组件

- **DTOs**：数据传输对象，用于封装从API传输到客户端的数据。

- **Commands**、**Events**、**Requests**：处理应用程序内部的命令、事件和请求的定义，这些通常涉及业务操作的触发和处理。

- **Responses**：定义API或服务对外部请求的响应结构。

- **MiddleWares**：中间件组件，处理HTTP请求的流，如身份验证、错误处理等。

- **Extensions**：扩展方法集，常用于跨多个模块共享的功能，如HTTP上下文、依赖注入容器配置等。

- **Attributes**：自定义属性，可能用于控制访问、路由配置等。

- **Bo**：业务对象，用于在业务层和表示层之间传递数据，可以视具体情况决定放置于 Core 或 Message。

  

# 参考考核项目时遇到的问题

## 1.问题一：The Email field is required！

![image-20240411091011695](assets/image-20240411091011695.png)

问题描述：在我的数据库表中，email的值明明可以为空，为什么提交请求时却要求必须有值呢？

![image-20240411091210504](assets/image-20240411091210504.png)

这个问题涉及到**数据绑定**的内容

### `[ApiController]` 的行为

在ASP.NET Core中使用 `[ApiController]` 属性时，该属性启用了一些默认行为，这些行为旨在简化API的开发和提高代码的一致性与可维护性。

![image-20240411091837590](assets/image-20240411091837590.png)

而在模型UserDto中，可以看到Email标着下黄波浪线，Non-nullable property 'Email' is uninitialized. Consider declaring the property as nullable.意思是不可为 null 的属性“电子邮件”未初始化。 考虑将该属性声明为可为空。![image-20240411091953287](assets/image-20240411091953287.png)

当控制器类上标注了 `[ApiController]` 属性，ASP.NET Core会自动验证传入的数据模型。如果模型不满足验证规则（比如数据注解要求的`[Required]`、`[Range]`、`[EmailAddress]`等），则会自动生成一个400 Bad Request响应。

简单来说，就是如果控制器中加了`[apicontroller]`，而传的参数又是**引用类型**时，则必须要为它添加一个默认值，或者把它设置为可空。

### 引用类型参数必须有值的原因

当你的控制器方法接受一个复杂类型的参数（比如一个类的实例），并且该参数是从请求体中绑定的，`[ApiController]` 属性会要求这个参数必须在请求体中存在。如果请求中没有提供这个参数的任何数据，模型绑定过程将无法创建这个参数的实例，因此无法进行进一步的验证，导致验证失败。

### 解决办法

#### 办法一

把email设置为可空：在`string`后面加一个？，表示email可空。

![image-20240411092315162](assets/image-20240411092315162.png)

此时我提交的json字符串：{ 

​    `"userDto":` 

​    `{`

​         `"username": "keria1",` 

​         `"password": "123"`

​    `}` 

`}`

可以看到，数据被成功的添加了进去![image-20240411092913310](assets/image-20240411092913310.png)



#### 办法 二

给email设置默认值

![image-20240411093208529](assets/image-20240411093208529.png)

此时我提交的json字符串

`{` 

​    `"userDto":` 

​    `{`

​         `"username": "keria2",` 

​         `"password": "123"`

​    `}` 

`}`

可以看到，数据被成功的添加了进去

![image-20240411093306025](assets/image-20240411093306025.png)

#### 办法三

在构造函数中设置默认值

![image-20240411093452725](assets/image-20240411093452725.png)

此时我的提交的json字符串

`{` 

​    `"userDto":` 

​    `{`

​         `"username": "keria3",` 

​         `"password": "123"`

​    `}` 

`}`

可以看到数据也被成功的添加了进去

![image-20240411093539455](assets/image-20240411093539455.png)



# ASP.NET Core 数据绑定特性

在ASP.NET Core中，有多种特性可以用来指定HTTP请求中数据的来源。这些特性帮助开发者明确每个参数应如何被解析和绑定，从而使控制器代码更清晰、易于维护。

## 1. FromQuery 特性

- **用途**：从请求的查询字符串中提取参数值。
- **常用场景**：通常用于GET请求，这些请求的参数通过查询字符串传递。
- **示例**：
  
  ```csharp
  [HttpGet]
  public IActionResult GetData([FromQuery] string parameter1, [FromQuery] int parameter2) {
      // 访问 /api/data?parameter1=keria777&parameter2=777
  }

## 2. FromBody 特性

- **用途**：从请求体中提取参数值。

- **常用场景**：用于POST、PUT、DELETE等请求，这些请求通过请求体传递参数。

- 示例

  ：

  ```
  csharpCopy code
  [HttpPost]
  public IActionResult PostData([FromBody] SomeModel model) {
      // 请求体包含 JSON: { "property1": "keira777", "property2": 777 }
  }
  ```

## 3. FromRoute 特性

- **用途**：从请求的URL路由中提取参数值。

- **常用场景**：适用于RESTful API，参数直接嵌入在URL中。

- 示例

  ：

  ```
  csharpCopy code
  [HttpGet("api/{id}")]
  public IActionResult GetById([FromRoute] int id) {
      // 访问 /api/777
  }
  ```

## 4. FromHeader 特性

- **用途**：从请求的头部中提取参数值。

- **常用场景**：适用于需要从HTTP头部获取信息的场景，如认证。

- 示例

  ：

  ```
  csharpCopy code
  [HttpGet]
  public IActionResult GetByHeader([FromHeader] string authorization) {
      // 请求头包含 Authorization: Bearer tokenabc
  }
  ```

## 5. FromForm 特性

- **用途**：从表单数据中提取参数值。

- **常用场景**：用于POST请求中的表单提交。

- 示例

  ：

  ```
  csharpCopy code
  [HttpPost]
  public IActionResult PostFormData([FromForm] string username, [FromForm] string password) {
      // 表单数据为 username=keria&password=777
  }
  ```

## 6. FromServices 特性

- **用途**：从服务容器中自动注入服务实例。

- **常用场景**：适用于需要依赖注入服务实例的操作，如使用日志记录。

- 示例

  ：

  ```
  csharpCopy code
  [HttpGet]
  public IActionResult GetFromService([FromServices] ILogger<MyController> logger) {
      // 使用注入的ILogger<MyController>
  }
  ```

通过使用这些特性，开发者可以确保控制器方法的参数正确地绑定来自HTTP请求的各种数据，从而提高了应用的安全性和效率。