# 本地CI



![image-20241004095049310](assets/image-20241004095049310.png)



![image-20241004095110086](assets/image-20241004095110086.png)





如果 TeamCity 服务器和 Agent 容器运行在同一台主机上，可以尝试将 `localhost` 替换为 `host.docker.internal`：

```
docker run -d -e SERVER_URL="http://host.docker.internal:8111" --name="teamcity-agent-test" custom-teamcity-agent-test
```

`host.docker.internal` 是 Docker 提供的主机名，用于在 Docker 容器中访问主机的 IP 地址。





## GitVersion版本控制

![image-20241007192233131](assets/image-20241007192233131.png)

要实现以上效果。

拉取镜像：

```
docker pull gittools/gitversion:latest
```

运行镜像：

```
docker run --rm -v "/Users/keria/Desktop/myExamTopic/PractiseForKeria/PractiseForKeria":/repo gittools/gitversion:latest /repo
```

再在TeamCity上build step 配置一个Command Line ： script里填 `gitversion /output buildserver`





## Container Build

### docker.server.version这个配置有坑

1. 进入容器：

   ```
   docker exec -it teamcity-agent-test bash
   ```

2. 编辑 `buildAgent.properties` 文件，通常位于 `/data/teamcity_agent/conf/`目录下：

   ```
   vim /data/teamcity_agent/conf/buildAgent.properties
   ```

   > [!CAUTION]
   >
   > 注意这里是data下，不是opt下，给这玩意坑了**三个小时**

   ![image-20241008140737237](assets/image-20241008140737237.png)

   > [!NOTE]
   >
   > `buildAgent.properties` 文件在 TeamCity 构建代理中的作用是配置构建代理的属性和设置。这两个不同位置的 `buildAgent.properties` 文件通常有以下区别：
   >
   > ### 1. **`/opt/buildagent/conf/buildAgent.properties`**
   >
   > - 这个文件是代理的默认配置文件，通常用于设置构建代理的基本属性，如：
   >   - `name`：代理的名称。
   >   - `serverUrl`：TeamCity 服务器的 URL。
   >   - `workDir`：代理的工作目录。
   >   - `tempDir`：临时文件夹。
   >   - `auth`：认证信息。
   > - 这个文件是 TeamCity 安装时提供的默认配置，通常用于标准的安装和配置。
   >
   > ### 2.**`/data/teamcity_agent/conf/buildAgent.properties`**
   >
   > - 这个文件通常是由代理在运行时创建的配置文件。它包含运行时生成的代理设置或通过代理的管理界面配置的设置。
   > - 这个文件可能会包含特定于代理实例的设置，比如一些运行时参数，或者用户在 TeamCity 界面上对代理进行的配置。
   >
   > ### 总结
   >
   > - **默认配置 vs. 运行时配置**：`/opt/buildagent/conf/buildAgent.properties` 是默认配置，而 `/data/teamcity_agent/conf/buildAgent.properties` 是运行时的配置，通常用于特定于代理的设置。
   > - **自定义设置**：在实际使用中，通常建议在运行的代理配置文件中（如 `/data/teamcity_agent/conf/buildAgent.properties`）进行更改，以确保在容器更新或重启时不会丢失配置。

3. 添加或修改以下行：

   ```
   docker.server.version=25.0.3
   ```

   ![image-20241008140209133](assets/image-20241008140209133.png)

   这个具体的版本号可以用 `docker info`命令查看，我的是25.0.3。

4. Esc 然后 :wq! 保存并退出。

5. 重启 Agent：

   在修改配置后，需要重启 TeamCity Agent 以使更改生效。可以使用以下命令重启容器：

   ```
   docker restart teamcity-agent-test
   ```

   ![image-20241008140120073](assets/image-20241008140120073.png)





# 本地CD

#### 1. 拉取 Octopus Deploy Docker 镜像

打开终端并运行以下命令以拉取最新的 Octopus Deploy 镜像：

```
docker pull octopusdeploy/octopusdeploy
```

在macOS 的 Apple Silicon（M1 或 M2）上使用 Docker 运行 Octopus Deploy 可能会遇到不兼容的问题，因为 Octopus Deploy 的官方 Docker 镜像主要为 x86_64 架构提供支持。以下是解决此问题的办法

**使用 Rosetta 2****

安装 Rosetta 2**： 打开终端并运行以下命令：

```
softwareupdate --install-rosetta
```

使用 Docker 的 `--platform` 参数指定使用 x86 架构：

没有配置settings就要加这一行指定--platform linux/amd64

设置 `ACCEPT_EULA` 环境变量来实现接受最终用户许可协议 (EULA)

```
docker run --name octopusserver -d \
  --platform linux/amd64 \
  -e ACCEPT_EULA=Y \
  -e OCTOPUS_DB_CONNECTION_STRING="Server=host.docker.internal;Database=PractiseForKeria;User Id=root;Password=123456;" \
  -e OCTOPUS_SERVER_BASE_URL="http://localhost:8080" \
  -p 8080:80 \
  octopusdeploy/octopusdeploy
```