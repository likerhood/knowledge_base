# maven使用

[TOC]



## **一、Maven 仓库**

==Maven 是一个 **Java 项目构建和依赖管理工具**==

Maven 中仓库用于存储项目依赖的构件（JAR、WAR 等）和插件，分为三类：

1. **本地仓库（Local Repository）**

   - **路径**：默认在用户目录下的 `.m2/repository`（如 `C:\Users\YourName\.m2\repository`）。
   - **作用**：缓存从远程仓库下载的依赖，避免重复下载。

2. **远程仓库（Remote Repository）**

   - **中央仓库（Central Repository）**：Maven 默认的公共仓库，包含大多数开源库（如 Spring、JUnit）。

   - 镜像仓库（Mirror）：加速访问的替代仓库（如阿里云镜像）。

     ```xml
     <!-- 在 settings.xml 中配置镜像 -->
     <mirror>
       <id>aliyun</id>
       <mirrorOf>*</mirrorOf>
       <name>Aliyun Mirror</name>
       <url>https://maven.aliyun.com/repository/public</url>
     </mirror>
     ```

   - **私有仓库（Private Repository）**：企业自建的仓库（如 Nexus、Artifactory），用于存储内部依赖。

3. **仓库优先级**
   Maven 按以下顺序查找依赖：
   ​**本地仓库 → 私有仓库 → 镜像仓库 → 中央仓库**。



## **二、项目结构**
Maven 强制约定目录结构，确保标准化：
```
my-project
├── src
│   ├── main
│   │   ├── java      # 主代码
│   │   └── resources # 配置文件（如 XML、properties）
│   └── test
│       ├── java      # 测试代码
│       └── resources # 测试配置
├── target            # 构建输出目录（编译后的文件、JAR 包）
└── pom.xml           # 项目配置文件
```



## 三、Maven 核心概念

1. **POM（Project Object Model）**
   Maven 项目的配置文件是 `pom.xml`，**用于管理java项目及其依赖**，定义了：

   - 项目坐标（`groupId`, `artifactId`, `version`）
   - 依赖（`dependencies`）
   - 构建配置（如插件、资源目录等）

2. **坐标（Coordinates）**
   通过 `groupId`（组织标识）、`artifactId`（项目名）、`version`（版本）**唯一标识一个项目或依赖**，例如：

   ```xml
   <groupId>com.example</groupId>
   <artifactId>my-project</artifactId>
   <version>1.0.0</version>
   ```

3. **仓库（Repository）**

   - **本地仓库**：位于用户目录下的 `.m2/repository`，缓存已下载的依赖。
   - **远程仓库**：如 Maven 中央仓库（Central Repository）、公司私有仓库（Nexus/Artifactory）。

4. **生命周期（Lifecycle）**
   Maven 定义了构建流程的默认生命周期阶段（如 `compile`, `test`, `package`, `install`, `deploy`），每个阶段对应一组插件目标（Plugin Goals）。



## 四、核心命令

通常idea右侧有maven的项目工具栏，可视化按钮来操作。

| 命令                  | 作用                                 |
| --------------------- | ------------------------------------ |
| `mvn clean`           | 删除 `target` 目录                   |
| `mvn compile`         | 编译主代码                           |
| `mvn test`            | 运行单元测试                         |
| `mvn package`         | 打包项目（生成 JAR/WAR 到 `target`） |
| `mvn install`         | 打包并安装到本地仓库                 |
| `mvn deploy`          | 部署到远程仓库（需配置）             |
| `mvn dependency:tree` | 查看依赖树（解决冲突）               |

**常用组合：**  

```bash
mvn clean package                 # 清理后打包
mvn clean install -DskipTests     # 跳过测试并安装到本地仓库
```





## **五、多模块项目配置**

1. **父项目（Parent Project）**

   - **作用**：统一管理子模块的公共配置（如依赖版本、插件）。

   - 配置：

     ```xml
     <!-- 父项目 pom.xml -->
     <packaging>pom</packaging>
     <modules>
       <module>module-1</module>
       <module>module-2</module>
     </modules>
     ```

2. **子模块（Submodules）**

   - 继承父配置

     ```xml
     <!-- 子模块 pom.xml -->
     <parent>
       <groupId>com.example</groupId>
       <artifactId>parent-project</artifactId>
       <version>1.0.0</version>
     </parent>
     ```

   - **独立配置**：子模块可覆盖父项目的配置。

3. **聚合构建**
   在父目录执行 `mvn clean install`，Maven 会按顺序构建所有子模块。

**总结：**

- **父项目**：
  - 管理子模块列表和公共配置。
  - 通过 ` <dependencyManagement> ` 统一依赖版本。
- **子模块**：
  - 继承父项目的配置。
  - 声明自己的依赖和插件。
- **构建顺序**：
  Maven 根据模块依赖关系自动确定构建顺序，确保依赖模块先构建。
- **本地仓库**：
  子模块的构件（如 `ecommerce-core-1.0.0.jar`）会被安装到本地仓库，供其他模块或项目使用。

