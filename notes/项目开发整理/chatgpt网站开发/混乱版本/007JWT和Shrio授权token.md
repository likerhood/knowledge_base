# JWT和Shrio实现token鉴权

[TOC]



## 1. 核心诉求

以用户使用 OpenAI 接口，如；http://localhost/api 时，需要根据用户身份标识做一些访问的验证和限定。最直接就是在使用 api 的时候把用户的账号和密码一同和访问 api 传递过来，如；http://localhost/api?userId=xfg&password=123 但这样就把用户的密码信息给泄漏了，是非常不安全的。

所以**需要根据用户的账密，先通过登录验证的方式，发放一个 token**，之后用户再使用这个 token 配置到链接后面使用。如；http://localhost/api?token=xxxxx 这样就安全多了。—— 一般 token 是配置到 http 请求头信息中，但这里为了更加方便用户传递参数，所以这样处理了。



![1742979451139](img/1742979451139.png)







**Token 鉴权流程**关键点：

| 步骤          | 技术实现              | 安全目标         |
| :------------ | :-------------------- | :--------------- |
| 1. 用户登录   | 账密验证 → 生成 Token | 避免明文传输密码 |
| 2. API 访问   | 携带 Token 鉴权       | 防止未授权访问   |
| 3. Token 管理 | JWT（无状态令牌）     | 支持分布式验证   |
| 4. 权限控制   | Shiro Realm + Filter  | 灵活控制访问权限 |

```mermaid
sequenceDiagram
    participant User
    participant Nginx
    participant SpringBoot
    participant OpenAI

    User->>SpringBoot: 登录（账密）
    SpringBoot->>SpringBoot: 验证账密 → 生成JWT
    SpringBoot-->>User: 返回Token
    User->>Nginx: 访问API（?token=xxx）
    Nginx->>SpringBoot: 子请求验证Token
    alt Token有效
        SpringBoot-->>Nginx: 200 OK
        Nginx->>OpenAI: 转发请求
    else Token无效
        SpringBoot-->>Nginx: 401 Unauthorized
        Nginx-->>User: 拒绝访问
    end
```



## 2. 核心技术栈

### **2.1 JWT (JSON Web Token)**

- **作用**：生成无状态令牌，包含用户身份信息（如用户ID）。
- **代码中**：
  - `JwtToken`：封装 Token 的载体对象。
  - `JwtUtil`：负责 Token 的生成/解析/验证。

**无状态令牌**意味着服务端 **不需要存储会话信息**（如Session），所有必要的用户数据（如用户ID、权限）都直接编码在令牌本身中。 



### **2.2 Apache Shiro**

- **作用**：轻量级安全框架，处理认证（Authentication）和授权（Authorization）。
- **代码中**：
  - `JwtRealm`：继承 `AuthorizingRealm`，自定义 Token 验证逻辑。
  - `JwtFilter`：拦截请求，提取 Token 并交给 Shiro 验证。
  - `ShiroConfig`：配置 Shiro 的过滤器和 Realm。



## 2.3 token、cookie、session知识

[参考博客](https://hengheng.blog.csdn.net/article/details/107153309)

**Http是无状态的，有会话的**
**浏览器访问服务器时，浏览器和服务器之间就建立了连接**，连接后浏览器可以向服务器发送多次请求，并且这多次请求之间是没有任何关系的，从**服务器的角度而言，即便同一个浏览器像我发送了多次请求，我也不认为这些请求之间有什么关系，仍然把每次请求当做陌生人俩对待**。他不对之前的请求和响应状态进行管理，无法根据之前的状态对本次请求进行处理。



所以我们登录进去一个网站，后面我们的操作都希望服务器知道我们已经登录好，不能每一次操作都先登陆，之后下一步操作服务器就不知道你是谁。



那么token、cookie、session这些相关技术实现这一个部分。



**cookie：**

1. 存储在浏览器的一小块数据，**第一次连接好网站之后，浏览器会存储相关cookie**；
2. cookie没有设置生存时间，就存储在浏览器内存，浏览器关闭后cookie消失，设置生存时间，cookie会存储在硬盘；
3. **cookie相当于明文显示用户信息，有安全风险——例如黑客拦截cookie，恶意获取用户信息**

![1743008661716](img/1743008661716.png)



**session：**

1. 结合cookie使用；
2. 安全性有提升——**cookie存储的是session的id，用户信息在session里面**
3. **session存储在服务器中，外界很难获取用户信息**
4. **session机制难以实现分布式部署**
   1. **session存储在服务器内存中，服务器故障session消失；**
   2. 分布式部署指 **多个服务器分担用户量，提供服务；**
   3. 解决方式有：**将session对应的session_id和对应的用户名存储到数据库或者redis缓存（开销大）**

![1743009039799](img/1743009039799.png)





**token：**

1.  token一般翻译成令牌，一般是用于**验证用户身份的数据**；
2. **可以用url传参，也可以用post提交，也可以夹在http的header中**。它相当于session中的session_id，是一个字符串，**存放在浏览器**中；
3. token由后端产生，通过密钥和加密算法将用户信息加密到token中，同时也是通过密钥来验证token是否正确

![1743009174393](img/1743009174393.png)





## 3. 语法详解

### 3.1 生成token和验证权限

流程如下：

```mermaid
sequenceDiagram
    participant Client as 浏览器
    participant Server as 服务器
    participant DB as 数据库

    Client->>Server: 登录（username+password）
    Server->>DB: 验证账号密码
    DB-->>Server: 返回user_id=123
    Server->>Server: 生成JWT（含user_id, username）
    Server-->>Client: 返回Token

    Client->>Server: 访问主页（携带Token）
    Server->>Server: 验证Token签名+过期时间
    Server->>DB: 根据user_id查询用户详情（昵称、头像等）
    DB-->>Server: 返回用户数据
    Server-->>Client: 返回主页HTML（含用户数据）
```









### 3.2 shrio整合JWT

 JWT本质上就是一串加密的字符串，JWT和Shiro的整合就是让JwtFilter截获用户请求头中的token，如果有token就交给Shiro的自定义Realm去判断token是否正确，是否过期等。 

```mermaid
sequenceDiagram
    participant Client
    participant JwtFilter
    participant JwtRealm
    participant Controller

    Client->>JwtFilter: 请求 /api/xxx?token=eyJ...
    JwtFilter->>JwtRealm: subject.login(jwtToken)
    JwtRealm->>JwtRealm: 验证签名/过期时间
    alt 验证成功
        JwtRealm-->>JwtFilter: 返回AuthenticationInfo
        JwtFilter-->>Controller: 放行请求
        Controller-->>Client: 返回业务数据
    else 验证失败
        JwtRealm-->>JwtFilter: 抛出异常
        JwtFilter-->>Client: 返回401错误
    end
```

#### 3.2.1 shiro配置文件

`shirioConfig.java`**文件用于指定那些路径需要去验证token和鉴别权限**

##### **1. SubjectFactory**

- **作用**：控制 Subject 实例的创建过程

- **Subject是什么：** `Subject` 是 Apache Shiro 的核心概念，代表 **当前执行用户**（可以是人、服务、设备等）。它封装了所有与安全相关的交互， 

- **2. Subject 的核心功能**

  | **方法**                     | **作用**                                 | **示例**                                                  |
  | :--------------------------- | :--------------------------------------- | :-------------------------------------------------------- |
  | `login(AuthenticationToken)` | 提交凭证（如用户名密码/JWT）进行认证     | `subject.login(new UsernamePasswordToken("user", "123"))` |
  | `isAuthenticated()`          | 检查是否已认证                           | `if(subject.isAuthenticated()){...}`                      |
  | `hasRole(String role)`       | 检查是否拥有指定角色                     | `subject.hasRole("admin")`                                |
  | `isPermitted(String perm)`   | 检查是否拥有指定权限                     | `subject.isPermitted("user:delete")`                      |
  | `getPrincipal()`             | 获取身份标识（如用户名/userId）          | `String username = (String) subject.getPrincipal()`       |
  | `getSession()`               | 获取关联的 Session（无状态架构可能禁用） | `Session session = subject.getSession()`                  |
  | `logout()`                   | 注销当前用户                             | `subject.logout()`                                        |

- **关键方法**：

  ```
  context.setSessionCreationEnabled(false); // 禁用Session
  ```

- **设计意义**：确保创建的 Subject 不会持有 Session 引用

##### **2. SessionStorageEvaluator**

- **功能**：决定是否存储 Session 数据

- **关闭存储**：

  ```
  evaluator.setSessionStorageEnabled(false);
  ```

- **影响**：`subject.getSession()` 将返回空或抛出异常

##### **3. FilterChain 执行流程**

```mermaid
graph TD
	A[请求进入] --> B{URL匹配}
    B -->|/login| C[AnonymousFilter:直接放行]
    B -->|/verify| D[JwtFilter]
    D --> E[提取Token]
    E --> F[提交Realm验证]
    F -->|成功| G[执行业务逻辑]
    F -->|失败| H[返回401]
    B -->|/logout| I[LogoutFilter:清理状态]
```

##### 

```java
@Configuration
public class ShiroConfig {

    @Bean
    public SubjectFactory subjectFactory() {
        class JwtDefaultSubjectFactory extends DefaultWebSubjectFactory{
            @Override
            public Subject createSubject(SubjectContext context) {
                context.setSessionCreationEnabled(false);
                return super.createSubject(context);
            }
        }
        return new JwtDefaultSubjectFactory();
    }

    @Bean
    public Realm realm() {
        return new JwtRealm();
    }

    @Bean
    public DefaultWebSecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(realm());
        // 关闭 ShiroDAO 功能
        DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
        DefaultSessionStorageEvaluator defaultSessionStorageEvaluator = new DefaultSessionStorageEvaluator();
        // 不需要将 Shiro Session 中的东西存到任何地方（包括 Http Session 中）
        defaultSessionStorageEvaluator.setSessionStorageEnabled(false);
        subjectDAO.setSessionStorageEvaluator(defaultSessionStorageEvaluator);
        securityManager.setSubjectDAO(subjectDAO);
        // 禁止Subject的getSession方法
        securityManager.setSubjectFactory(subjectFactory());
        return securityManager;
    }

    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean() {
        ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
        shiroFilter.setSecurityManager(securityManager());
        shiroFilter.setLoginUrl("/unauthenticated");
        shiroFilter.setUnauthorizedUrl("/unauthorized");
        // 添加jwt过滤器
        Map<String, Filter> filterMap = new HashMap<>();
        // 设置过滤器【anon\logout可以不设置】
        filterMap.put("anon", new AnonymousFilter());
        filterMap.put("jwt", new JwtFilter());
        filterMap.put("logout", new LogoutFilter());
        shiroFilter.setFilters(filterMap);

        // 拦截器，指定方法走哪个拦截器 【login->anon】【logout->logout】【verify->jwt】
        Map<String, String> filterRuleMap = new LinkedHashMap<>();
        filterRuleMap.put("/login", "anon");
        filterRuleMap.put("/logout", "logout");
        filterRuleMap.put("/verify", "jwt");
        shiroFilter.setFilterChainDefinitionMap(filterRuleMap);

        return shiroFilter;
    }

}

```





#### 3.2.2 JwtToken.java表示token对象

在JWT没有和Shiro整合之前，用户的账号密码被封装成了其自带的UsernamePasswordToken对象。UsernamePasswordToken 其实是 AuthenticationToken 的实现类。 

 **Shiro 的安全认证体系要求所有身份凭证（token）必须封装为 `AuthenticationToken` 对象。**

- **`getPrincipal()`**：返回用户身份标识（通常为用户名或用户ID）
- **`getCredentials()`**：返回验证凭证（通常为密码或Token）

```java
import org.apache.shiro.authc.AuthenticationToken;

public class JwtToken implements AuthenticationToken {

    private String jwt;

    public JwtToken(String jwt) {
        this.jwt = jwt;
    }

    /**
     * 等同于账户
     */
    @Override
    public Object getPrincipal() {
        return jwt;
    }

    /**
     * 等同于密码
     */
    @Override
    public Object getCredentials() {
        return jwt;
    }

}

```





#### 3.2.3 JWTFilter过滤器取出token

在Shiro中，ShiroFilter用来过滤所有的请求，一般情况下shiro通过传入用户名和密码生成UsernamePasswordToken后使用subject.login进行登录。

但是这里我们整合了JWT和Shiro，**所以要自定义一个过滤器JWTFilter**。它的主要作用是拦**截请求，判断请求头中是否含有token，如果有就交给Realm进行鉴权处理。**

**AccessControlFilter抽象类**里面有两个必须要实现的方法：**isAccessAllowed()方法和onAccessDenied()**方法；

在执行登录的时候会调用AccessControlFilter类里面的onPreHandle方法，所有的请求经过过滤器都会来到onPreHandle方法，该方法会自动调用下面两个两个方法决定是否继续处理：

onPreHandle()方法：

- isAccessAllowed：**判断是否登录，在登录的情况下会走此方法**
- onAccessDenied：**是否是拒绝登录，没有登录的情况下会走此方法**
- 如果isAccessAllowed方法返回True，则不会再调用onAccessDenied方法，代表已经登录；
- 如果isAccessAllowed方法返回Flase，则会继续调用onAccessDenied方法。而onAccessDenied方法里面则是具体执行登陆的地方。



```java
public class JwtFilter extends AccessControlFilter {

    private Logger logger = LoggerFactory.getLogger(JwtFilter.class);

    /**
     * isAccessAllowed 判断是否携带有效的 JwtToken
     * 所以这里直接返回一个 false，让它走 onAccessDenied 方法流程
     */
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        return false;
    }

    /**
     * 返回结果为true表明登录通过
     */
    @Override
    protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        // 如果你设定的 token 放到 header 中，则可以这样获取；request.getHeader("Authorization");
        JwtToken jwtToken = new JwtToken(request.getParameter("token"));
        try {
            // 鉴权认证
            getSubject(servletRequest, servletResponse).login(jwtToken);
            return true;
        } catch (Exception e) {
            logger.error("鉴权认证失败", e);
            onLoginFail(servletResponse);
            return false;
        }
    }

    /**
     * 鉴权认证失败时默认返回 401 状态码
     */
    private void onLoginFail(ServletResponse response) throws IOException {
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        httpResponse.getWriter().write("Auth Err!");
    }

}

```



#### 3.2.4 JWTRealm

1. ##### **`doGetAuthorizationInfo()` - 授权逻辑**

2. ##### **`doGetAuthenticationInfo()` - 认证逻辑**

   - **参数解析**：
     - `token`：由`JwtFilter`传入的`JwtToken`对象
   - **验证流程**：
     1. **空值检查**：拒绝无Token请求
     2. **密码学验证**：通过`JwtUtil`校验签名和过期时间
     3. **业务数据提取**：解码Payload获取用户信息（如username）

   | 参数          | 本例值     | 常规用法   | JWT特殊处理原因     |
   | :------------ | :--------- | :--------- | :------------------ |
   | `principal`   | JWT字符串  | 用户名     | JWT自身包含身份信息 |
   | `credentials` | JWT字符串  | 密码       | 无需二次验证        |
   | `realmName`   | "JwtRealm" | 自定义名称 | 用于多Realm区分     |

3. ##### **`supports()` - 类型支持声明**

```java
public class JwtRealm extends AuthorizingRealm {

    private Logger logger = LoggerFactory.getLogger(JwtRealm.class);

    private static JwtUtil jwtUtil = new JwtUtil();

    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JwtToken;// 只处理JwtToken类型
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        // 暂时不需要实现
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String jwt = (String) token.getPrincipal();
        if (jwt == null) {
            throw new NullPointerException("jwtToken 不允许为空");
        }
        // 判断
        if (!jwtUtil.isVerify(jwt)) {
            throw new UnknownAccountException();
        }
        // 可以获取username信息，并做一些处理
        String username = (String) jwtUtil.decode(jwt).get("username");
        logger.info("鉴权用户 username：{}", username);
        return new SimpleAuthenticationInfo(jwt, jwt, "JwtRealm");
    }

}

```





#### 3.2.5 JTUtils.java工具类

```java

```







### 4. 验证

1. 授权Token：**http://localhost:8080/authorize?username=xfg&password=123** - 服务启动，访问地址，获取Token



![img](img/Frf8GbkJseWEGXi-xUPr7YXDPSnc)



1. 使用Token：**http://localhost/api/?token=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ4ZmciLCJleHAiOjE2ODMxMDQ4ODcsImlhdCI6MTY4MzEwNDU4NywianRpIjoiYzg5Y2YyZTQtODdiMy00ZjcyLWE0NTUtNTA4YTVkZjQxMTViIiwidXNlcm5hbWUiOiJ4ZmcifQ.Vy5QTZ8iDzOt-fGd_NKrT_IOBce8mBEx0gV0o0zHfaY** - 把上图中的 Token 复制到这里就可以使用了。



![img](img/FubdOEt0F1lInn3PtwHCoVE7KPcD)



1. Token 可以压缩，缩短字符串。你可以尝试下做这个优化处理。
2. 这个 Token 也就是一般付费模式中，购买的东西，购买后因为本身都是在一个系统下登录的话，那么用户是感知不到的。但实际上就是这么一个东西来驱动流程执行。

























