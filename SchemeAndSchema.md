“Scheme”和“Schema”是两个在编程和数据库领域中常见的术语，虽然它们的拼写相似，但它们在概念和使用上有显著区别。以下是它们的定义和区别：

### Scheme（方案）

1. **定义**：在编程和网络中，"scheme" 指的是一种用于标识和区分不同资源的访问协议。例如，在URL中，"http" 和 "https" 是两种不同的 scheme。

2. 使用场景

   - **URL中的Scheme**：URL中的scheme指定了访问资源的方法。例如，`http://example.com` 中的 `http` 是一个 scheme，它表明该资源使用 HTTP 协议访问。
   - **身份验证中的Scheme**：在身份验证中，scheme可以指不同的认证方法。例如，在ASP.NET Core中，你可以定义多个身份验证scheme来处理不同的认证策略。
   - **编程语言**：Scheme 也是一种函数式编程语言，属于 Lisp 家族。

### Schema（模式）

1. **定义**：在数据库和数据建模中，"schema" 指的是数据库结构的定义，描述了数据库中数据的组织和约束。它包括表、视图、索引、存储过程等。

2. 使用场景

   - **数据库Schema**：数据库schema描述了数据库的逻辑结构，包括表的名称、字段名称和类型、关系等。例如，SQL中的 CREATE SCHEMA 语句用于定义一个新的数据库模式。
   - **XML Schema**：用于定义XML文档结构的规范，通常使用XSD（XML Schema Definition）。
   - **JSON Schema**：用于定义JSON数据结构的规范，可以验证JSON数据的格式和类型。

### 举例说明

#### Scheme

1. **URL中的Scheme**：

   - `http://www.example.com`
   - `https://www.example.com`
   - `ftp://ftp.example.com`

2. **身份验证中的Scheme**（例如在ASP.NET Core中）：

   ```c#
   services.AddAuthentication(options =>
   {
       options.DefaultAuthenticateScheme = "Cookies";
       options.DefaultChallengeScheme = "Cookies";
   })
   .AddCookie("Cookies", options =>
   {
       options.LoginPath = "/Account/Login";
   })
   .AddJwtBearer("Bearer", options =>
   {
       options.TokenValidationParameters = new TokenValidationParameters
       {
           ValidateIssuer = true,
           ValidateAudience = true,
           ValidateLifetime = true,
           ValidateIssuerSigningKey = true,
           // 设置其他参数
       };
   });
   ```

#### Schema

1. **数据库Schema**（例如在SQL中）：

   ```c#
   CREATE SCHEMA Sales;
   
   CREATE TABLE Sales.Orders
   (
       OrderID int,
       OrderDate datetime,
       CustomerID int,
       Amount decimal(10, 2)
   );
   ```
   
2. **JSON Schema**：

   ```json
   {
       "$schema": "http://json-schema.org/draft-07/schema#",
       "title": "Person",
       "type": "object",
       "properties": {
           "firstName": {
               "type": "string"
           },
           "lastName": {
               "type": "string"
           },
           "age": {
               "description": "Age in years",
               "type": "integer",
               "minimum": 0
           }
       },
       "required": ["firstName", "lastName"]
   }
   ```

### 总结

- **Scheme** 通常用于描述一种访问协议或方法，如URL中的协议部分或身份验证中的认证方法。

  侧重于方法和协议

- **Schema** 则用于描述数据的结构和组织，如数据库模式或数据交换格式（如XML和JSON）的定义。

理解这两个概念的区别可以帮助你在编程和数据建模中更准确地使用和理解它们。

​	侧重于数据结构和约束







### 类固定装置（Class Fixture）

#### 何时使用

当您希望在测试类的所有测试之间共享单个测试上下文，并在所有测试完成后清理它时使用。

#### 原因

创建和清理测试上下文可能非常昂贵，如果每次测试都要重复这些操作，会使测试运行速度变慢。通过使用类固定装置，可以在所有测试之间共享一个上下文实例，从而提高测试效率。





### 装配固定装置（Assembly Fixture）

#### 何时使用

当您希望在整个测试程序集中的所有测试之间共享单个测试上下文，并在所有测试完成后清理它时使用。

#### 原因

有时需要在整个测试程序集范围内共享一个固定对象，例如共享一个数据库连接。装配固定装置可以在整个测试程序集之间共享一个上下文实例。