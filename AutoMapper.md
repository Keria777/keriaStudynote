# AutoMapper 使用指南

AutoMapper 是一个在 .NET 应用中广泛使用的对象到对象映射库，它可以自动化对象之间的转换过程，从而简化数据模型之间的转换和数据操作。

## 安装

首先，通过 NuGet 安装 AutoMapper：

```c#
Install-Package AutoMapper
```

## 基本用法

### 模型定义

定义两个模型，一个是实体模型 `User`，另一个是数据传输对象模型 `UserDTO`。

```c#
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

public class UserDTO
{
    public string Name { get; set; }
    public string Email { get; set; }
}
```

### 配置映射

在 ASP.NET Core 的 `Startup.cs` 中配置 AutoMapper。

```c#
public void ConfigureServices(IServiceCollection services)
{
    var mappingConfig = new MapperConfiguration(mc =>
    {
        mc.AddProfile(new MappingProfile());
    });

    IMapper mapper = mappingConfig.CreateMapper();
    services.AddSingleton(mapper);
}

public class MappingProfile : Profile
{
    public MappingProfile()
    {
        CreateMap<User, UserDTO>();
        CreateMap<UserDTO, User>();
    }
}
```

### 使用映射

在控制器中使用 AutoMapper 进行对象映射。

```c#
public class UsersController : ControllerBase
{
    private readonly IMapper _mapper;

    public UsersController(IMapper mapper)
    {
        _mapper = mapper;
    }

    [HttpGet]
    public ActionResult<UserDTO> GetUser(int id)
    {
        User user = GetUserFromDatabase(id); // 假设这是从数据库获取用户的方法
        UserDTO userDto = _mapper.Map<UserDTO>(user);
        return Ok(userDto);
    }
}
```

## 高级功能

### 条件映射

**用法**： 条件映射允许在满足特定条件时才执行映射，只希望在某些字段满足特定条件时更新时非常有用。

**场景示例**： 在用户更新其信息时，可能只想更新那些实际更改了的字段。例如，只想更新那些已验证的电子邮件地址。

```c#
CreateMap<User, UserDTO>()
    .ForMember(dest => dest.Email, opt => opt.Condition(src => src.Email.Contains("@example.com")));
```

在这个例子中，只有当源 `Email` 字段包含 `@example.com` 时，才会将该 `Email` 映射到目标 `UserDTO` 对象。

### 自定义值解析器

**用法**：为复杂的映射逻辑编写自定义解析器。

**场景示例**：在一个用户管理系统中，需要展示的用户信息不仅包括姓名，还要包括一个唯一标识符（ID）。为了方便地识别和引用，我们希望在显示用户信息时，将用户的 `Id` 和 `Name` 组合成一个字符串。

```c#
public class CustomResolver : IValueResolver<User, UserDTO, string>
{
    public string Resolve(User source, UserDTO destination, string destMember, ResolutionContext context)
    {
      // 将用户的 ID 和姓名合并为一个新的字符串，中间用 " - " 分隔
        return source.Id + " - " + source.Name;
    }
}

// 在 AutoMapper 的配置中使用 CustomResolver 来映射 User 的 Name 字段
CreateMap<User, UserDTO>()
    .ForMember(dest => dest.Name, opt => opt.MapFrom<CustomResolver>());
```

这里，Name 字段被映射为用户的 Id 和 Name 的组合。

### 嵌套映射

**用法**：嵌套映射用于自动处理一个对象内嵌套其他对象或对象集合的情况。Automapper 能够识别这些内嵌的对象，并应用相应的映射规则，无需手动编写复杂的映射逻辑。

**场景示例**：在一个电商平台中，客户的订单信息是嵌套在客户信息中的。每个客户（`Customer`）都有一个地址（`Address`），这个地址需要单独映射到客户的数据传输对象中。

```c#
public class Customer
{
    public string Name { get; set; }
    public Address Address { get; set; }
}

public class CustomerDTO
{
    public string Name { get; set; }
    public AddressDTO Address { get; set; }
}

CreateMap<Address, AddressDTO>();
CreateMap<Customer, CustomerDTO>();
```

在这个例子中，`Customer` 类中嵌套的 `Address` 类自动映射到 `CustomerDTO` 中的 `AddressDTO` 类。这样做简化了代码，保持了模型之间的清晰界限投影

### 投影

**用法**：投影功能允许在使用 LINQ 查询时直接应用定义好的映射配置。这样可以直接在数据库查询时生成 DTOs，而不需要先加载实体到内存中再转换。

**场景示例**：在实现一个 API 接口时，需要从数据库中提取用户信息并直接返回用户的 DTO。使用投影可以直接在数据库层面转换数据，减少内存和CPU的使用。

```c#
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
}

public class UserDTO
{
    public string Name { get; set; }
}

// 在 API 控制器或服务层中
var userDTOs = dbContext.Users
    .Where(u => u.IsActive)
    .ProjectTo<UserDTO>(_mapper.ConfigurationProvider)
    .ToList();
```

这个例子展示了如何使用 AutoMapper 的投影功能，从数据库中直接转换并提取出 `UserDTO` 集合，这个操作是高效且资源友好的。

### 继承映射

应用继承结构的数据模型映射。

```c#
CreateMap<Person, PersonDTO>();
CreateMap<Employee, EmployeeDTO>().IncludeBase<Person, PersonDTO>();
```

**用法**：继承映射允许你定义基类到基类 DTO 的映射，并自动应用这些映射到派生类中，无需重复定义共通属性的映射。

**场景示例**：在一个处理多种类型员工的系统中，所有员工共享一些基本属性，如姓名和电子邮件，而某些特定类型的员工还有额外的属性，如部门或级别。

```c#
public class Employee
{
    public string Name { get; set; }
    public string Email { get; set; }
}

public class Manager : Employee
{
    public string Department { get; set; }
}

public class EmployeeDTO
{
    public string Name { get; set; }
    public string Email { get; set; }
}

public class ManagerDTO : EmployeeDTO
{
    public string Department { get; set; }
}

CreateMap<Employee, EmployeeDTO>();
CreateMap<Manager, ManagerDTO>().IncludeBase<Employee, EmployeeDTO>();
```

这个例子中，`Manager` 继承自 `Employee`，并且通过 `IncludeBase` 方法，Automapper 自动应用了从 `Employee` 到 `EmployeeDTO` 的映射规则到 `Manager` 到 `ManagerDTO` 的映射中，避免了重复代码并保持了映射的一致性。

## 实战

### 把对象A的属性C映射到对象B的属性D

1.创建实体

```c#
public class SourceA
{
    public int C { get; set; } // 假设C是一个int类型
}

public class DestinationB
{
    public int D { get; set; } // 假设D是一个int类型，映射目标
}
```

2.定义映射关系

```c#
var config = new MapperConfiguration(cfg => {    
  cfg.CreateMap<SourceA, DestinationB>()        
    .ForMember(dest => dest.D, opt => opt.MapFrom(src => src.C)); });
```

3.使用AutoMapper进行对象映射

```c#
var source = new SourceA { C = 100 };
var destination = mapper.Map<DestinationB>(source);
```

4.验证结果

```c#
Console.WriteLine(destination.D);  // 输出应该是100
```



### 把A对象映射到B对象，但是排除A对象的C属性

1.定义实体

```c#
public class SourceA
{
    public int C { get; set; }  // 要被忽略的属性
    public string X { get; set; }  // 其他属性
    public double Y { get; set; }
}

public class DestinationB
{
    public string X { get; set; }  // 相应的属性
    public double Y { get; set; }
}
```

2.配置映射关系

```c#
// 创建Mapper配置
var config = new MapperConfiguration(cfg => {
    cfg.CreateMap<SourceA, DestinationB>()
       .ForMember(dest => dest.X, opt => opt.MapFrom(src => src.X))
       .ForMember(dest => dest.Y, opt => opt.MapFrom(src => src.Y))
       .ForMember(src=>src.CreateAt,opt=>opt.Ignore());
});

// 使用配置创建Mapper对象
var mapper = config.CreateMapper();
```

3.使用AutoMapper进行对象映射

```c#
var source = new SourceA { C = 100, X = "Hello", Y = 3.14 };
var destination = mapper.Map<DestinationB>(source);
```

4.验证结果

```c#
Console.WriteLine($"X = {destination.X}, Y = {destination.Y}");  // 输出应该是 "X = Hello, Y = 3.14"
```

