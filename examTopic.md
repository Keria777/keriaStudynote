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