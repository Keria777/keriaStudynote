    public void ConfigureServices(IServiceCollection services)
    {
        // 配置服务
    }

**ConfigureServices 方法**

- `ConfigureServices` 方法是用来配置应用程序的服务的。在这个方法中，你可以注册应用程序所需的服务，比如数据库上下文、身份验证服务、MVC 中间件等。



    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
    
        app.UseRouting();
    
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapGet("/", async context =>
            {
                await context.Response.WriteAsync("Hello, world!");
            });
        });
    }

**Configure 方法**

- `Configure` 方法用于配置应用程序的请求处理管道。在这个方法中，你可以定义中间件的顺序和逻辑，以处理传入的 HTTP 请求。
- `env.IsDevelopment()` 检查当前应用程序是否在开发环境下运行。如果是开发环境，就启用开发者异常页面，方便调试。
- `app.UseRouting()` 启用路由中间件，用于处理传入的 HTTP 请求并将它们路由到适当的处理程序。
- `app.UseEndpoints()` 配置终结点路由，用于指定 URL 路由和相应的处理程序。在这个示例中，将根路径 `/` 映射到一个异步的请求处理程序，该处理程序简单地返回一个 "Hello, world!" 字符串。