# Hangfire

## 整合`.NET Core`项目小demo

- 以使用`MySQL`作为存储为例

1. 通过`NuGet`导入相关包

   ```c#
   <PackageReference Include="Hangfire" Version="1.8.10" />
   <PackageReference Include="Hangfire.MySqlStorage" Version="2.0.3" />
   ```

2. Startup中配置

   ```c#
   public class StartUp
   {
       public StartUp(IConfiguration configuration)
       {
           Configuration = configuration;
       }
   
       public IConfiguration Configuration { get; }
       
       public void ConfigureServices(IServiceCollection services)
       {
           services.AddControllersWithViews();
           
           // 配置 Hangfire 使用 MySQL 作为存储
           services.AddHangfire(config =>
               config.UseStorage(
                   new MySqlStorage("Server=localhost;Database=HangfireTest;User=root;Password=123456;", 
                       new MySqlStorageOptions
                       {
                           // 你可以在这里配置其他选项
                       })));
           
           // 添加 Hangfire 服务
           services.AddHangfireServer();
       }
       
       public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
       {
           if (env.IsDevelopment())
           {
               app.UseDeveloperExceptionPage();
           }
           
           app.UseHttpsRedirection();
           app.UseRouting();
           
           // 配置 Hangfire 仪表盘
           app.UseHangfireDashboard();
   
           // 调度一个每天下午4点20分执行的定时任务
           RecurringJob.AddOrUpdate<MyBackgroundJob>(job => job.Execute(), "20 16 * * *");
           
           app.UseAuthorization();
           
           // 其他中间件
           app.UseEndpoints(endpoints =>
           {
               endpoints.MapControllers();
           });
       }
   }
   ```

3. 具体逻辑

   ```c#
   public class MyBackgroundJob
   {
       public void Execute()
       {
           Console.WriteLine("后台任务正在执行...");
       }
   }	
   
   public class HomeController : Controller
   {
       // GET
       public string Index()
       {
           // 调度一个后台任务
           BackgroundJob.Enqueue<MyBackgroundJob>(job => job.Execute());
           return "success";
       }
   }
   ```

4. 调度Hangfire作业，并通过API断点来触发这些作业

   ```c#
   [Route("api/[controller]")]
   public class JobsController : ControllerBase
   {
       [HttpGet("fire-and-forget")]
       public IActionResult FireAndForget()
       {
           var jobId = BackgroundJob.Enqueue(() => Console.WriteLine("发射后不管！"));
           return Ok($"Fire-and-forget job enqueued. Job ID: {jobId}");
       }
   
       [HttpGet("delayed")]
       public IActionResult Delayed()
       {
           var jobId = BackgroundJob.Schedule(() => Console.WriteLine("延迟！"), TimeSpan.FromDays(7));
           return Ok($"Delayed job scheduled. Job ID: {jobId}");
       }
   
       [HttpGet("recurring")]
       public IActionResult Recurring()
       {
           RecurringJob.AddOrUpdate("我的重复作业", () => Console.WriteLine("重复！"), Cron.Daily);
           return Ok("Recurring job added or updated.");
       }
   
       [HttpGet("continuation")]
       public IActionResult Continuation()
       {
           var parentJobId = BackgroundJob.Enqueue(() => Console.WriteLine("父作业"));
           BackgroundJob.ContinueJobWith(parentJobId, () => Console.WriteLine("继续！"));
           return Ok($"Continuation job scheduled after job ID: {parentJobId}");
       }
   
       [HttpGet("batch")]
       public IActionResult Batch()
       {
           // 模拟批处理作业
           var jobId1 = BackgroundJob.Enqueue(() => Console.WriteLine("Job 1"));
           var jobId2 = BackgroundJob.Enqueue(() => Console.WriteLine("Job 2"));
           BackgroundJob.ContinueJobWith(jobId2, () => Console.WriteLine("All jobs completed."));
   
           return Ok("Simulated batch job started.");
       }
   }
   ```

5. 启动项目，访问/hangfire路径。逐个访问JobsController里的接口

   ![image-20240527194223521](assets/image-20240527194223521.png)

6.  数据库中多了很多表

   ![image-20240527194729099](assets/image-20240527194729099.png)



控制台输出：![image-20240527194246907](assets/image-20240527194246907.png)



## 持久性

Hangfire 会将需要执行的后台作业和定时任务存储到持久化存储中，然后使用轮询来检查这些任务是否需要执行。这种方式与传统的定时器实现方式有所不同，因为它能够确保在应用程序重启或崩溃后，Hangfire 仍然能够继续执行尚未完成的任务。

