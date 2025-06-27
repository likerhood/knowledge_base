

# 快速搭建一个简单的SpringBoot项目-详细步骤

[toc]





## 快速搭建一个简单的SpringBoot项目

>前言

- 本文章仅供大家参考，如果对大家有起到帮助的话可以点赞支持一下~ 
- 主要发布是为了本人以后能方便的搭建一个SpringBoot项目的框架！！！ 
- 源码路径在文章最下方！

### 第一步新建项目

1.选择Spring Initializr


![img](img/3f62eac82720769b3c280d3352491f72.png)

2.点击下一步


![img](img/64cbb2d2a78fc0b7585216484f52a279.png)


3.修改jdk的版本,再点击下一步 **注意！** ![img](img/425a256ef75501447cc22b32acc8ea5c.png)


![img](img/05a7b5f31f071ce626c445ef10ed1555.png)

4.选中Spring Web,再下一步


![img](img/a4292998d3a93438ca52107d5564eefc.png)

5.给项目文件命名，再点击完成


![img](img/ba3ca5236b27af4eee5721877c9e4322.png)

这样子就会生成一个项目，如下图所示


![img](img/733249c6c555ccbd52594379df8ae796.png)

下图中这些文件如果没有需要的情况下一般就直接删掉就好了！


![img](img/b0e3cf66c4425f66023f4c55976dc0c3.png)

### 第二步导入依赖

按照上面的步骤完成的打开pom.xml文件的配置依赖应该和我的是一样的！


![img](img/1c922b53608527a39e3f5abc64a195d7.png)

接着我们添加一些需要的依赖

SpringBoot项目需要提供一个接口去拿到数据所有在这里我们需要能连接数据库的配置

```java
<!--springboot+mybatis的依赖-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>
		<!--MySQL数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
		<!--druid数据库连接池依赖-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.2.8</version>
        </dependency>
		<!--Lombok依赖（可以配置也可以不用配置具体看自己）-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
```

### 第三步配置Application

新建一个application.yml文件 （使用aplication.properties也是可以的，只是本人一般使用.yml格式的）


![img](img/3806240e09549c2c743c1aa85b6681cc.png)

配置项目需要修改的端口号、datasource、mybatis。


![img](img/a76e1cc72034dc424ddd9c6ac58f7de5.png)

```java
server:
  #设置端口号
  port: 8081 #默认端口是8080
spring:
  datasource:
    #数据库用户名
    username: root
    #数据库用户密码
    password: 123456
    #serverTimezone=UTC 解决市区的报错 一般mysql是8.0以上的是必须配置这个
    #userUnicode=true&characterEncoding=utf-8 指定字符编码、解码格式
    url: jdbc:mysql://localhost:3306/metest?serverTimezone=UTC&userUnicode=true&characterEncoding=utf-8
    #设置驱动类
    driver-class-name: com.mysql.cj.jdbc.Driver
    #设置数据源
    type: com.alibaba.druid.pool.DruidDataSource

    #Spring Boot 默认是不注入这些属性值的，需要自己绑定
    #druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入
    #如果允许时报错  java.lang.ClassNotFoundException: org.apache.log4j.Priority
    #则导入 log4j 依赖即可，Maven 地址：https://mvnrepository.com/artifact/log4j/log4j
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500

# 配置mybatis
mybatis:
  #指定pojo扫描包位置让mybatis自动扫描到指定义的pojo包下
  type-aliases-package: com.me.test.pojo
  #指定位置扫描Mapper接口对应的XML文件 classpath:xml文件位置
  mapper-locations: classpath:mapper/*.xml
```

### 第四步创建需要的mapper、service、cotroller层

#### 创建需要的文件夹


![img](img/f9bb0db048dcd2948d200549fdf25e66.png)

#### 创建数据库

spl语句代码

```java
CREATE DATABASE /*!32312 IF NOT EXISTS*/`metest` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `metest`;

/*Table structure for table `userinfo` */

DROP TABLE IF EXISTS `userinfo`;

CREATE TABLE `userinfo` (
  `id` int(10) NOT NULL AUTO_INCREMENT,
  `username` varchar(30) NOT NULL,
  `password` varchar(30) NOT NULL,
  `authority` varchar(30) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

/*Data for the table `userinfo` */

insert  into `userinfo`(`id`,`username`,`password`,`authority`) values (1,'root','123456','admin'),(2,'me','123456','admin');
```


![img](img/d7050f6365f60dea34ccbe1ad30c633e.png)

IDEA连接上Mysql数据库（主要为了方便查看创建pojo类和对于的mapper.xml文件）


![img](img/9cc888d60dd95eb413b55a21b2d8c0ac.png)


![img](img/c0fe0e11dba816788d13fc21df2bde80.png)


![img](img/acb46b5e9811a65bc1bb25096a252023.png)

找到需要的数据库


![img](img/247f374cb08c693882a1df155123617e.png)


![img](img/26e41762d2ab854b99dc770ba1034e4d.png)

一般pojo类、mapper接口、service接口名字都是按照数据库中表的名字来创建的

#### 创建pojo类

```java
//使用@Data自动生成需要的get、set
@Data
//使用@AllArgsConstructor自动生成有参构造
@AllArgsConstructor
//使用@NoArgsConstructor自动生成无参构造
@NoArgsConstructor
public class userInfo {
   
    
    private Integer id;
    private String username;
    private String password;
    private String authority;
}
```


![img](img/976a9e650acd624988fc81d74f035c39.png)

#### 创建mapper接口

```java
@Repository
@Mapper
public interface UserInfoMapper {
   

    /**
     * 增加一条数据
     * @param userInfo 数据
     */
    void add(UserInfo userInfo);

    /**
     * 删除一条数据
     * @param id 被删除数据的id
     */
    void delete(Integer id);

    /**
     * 修改一条数据
     * @param userInfo 修改的数据
     */
    void update(UserInfo userInfo);

    /**
     * 根据id去查询一条数据
     * @param id 查询的id
     */
    UserInfo queryById(Integer id);

    /**
     * 查询全部数据
     * @return
     */
    List<UserInfo> queryAll();
}
```


![img](img/ec2176c7ac86d51c7dc6ad4542724176.png)

#### 创建对于mapper接口的xml文件

需要的mapper基本配置

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.me.test.mapper.UserInfoMapper">


</mapper>
```

对于接口中的方法在添加需要的增删改查功能（原配置代码有问题、目前已修改）

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.me.test.mapper.UserInfoMapper">

    <insert id="add" parameterType="UserInfo">
        insert into metest.userinfo (username, password, authority)
         values (#{username},#{password},#{authority});
    </insert>

    <delete id="delete" parameterType="Integer">
        delete from metest.userinfo where id = #{id};
    </delete>

    <update id="update" parameterType="UserInfo">
        update metest.userinfo set username=#{username},password=#{password},authority=#{authority}
        where id=#{id};
    </update>

    <select id="queryById" parameterType="Integer" resultType="UserInfo">
        select * from metest.userinfo where id=#{id};
    </select>

    <select id="queryAll" resultType="UserInfo">
        select * from metest.userinfo;
    </select>

</mapper>
```

图中爆红不用管这个是因为我配置了一个插件的原因，实际在运行时不影响效果！


![img](img/54133aa7fe088adac41eafb95018945b.png)

#### 创建service层


![img](img/0651419204671dae223ffa350af2af8b.png)


![img](img/0435c2b633d8d6fa9b610b3c1548fa95.png)

UserInfoService代码（其实其中的方法也就是Maper接口中拷贝来的）

```java
public interface UserInfoService {
   
    /**
     * 增加一条数据
     * @param userInfo 数据
     */
    void add(UserInfo userInfo);

    /**
     * 删除一条数据
     * @param id 被删除数据的id
     */
    void delete(Integer id);

    /**
     * 修改一条数据
     * @param userInfo 修改的数据
     */
    void update(UserInfo userInfo);

    /**
     * 根据id去查询一条数据
     * @param id 查询的id
     */
    UserInfo queryById(Integer id);

    /**
     * 查询全部数据
     * @return
     */
    List<UserInfo> queryAll();
}
```

UserInfoServiceImpl代码（主要是做业务逻辑的）

有需要添加的功能可以直接在这一层添加修改

```java
@Service
public class UserInfoServiceImpl implements UserInfoService {
   

    @Autowired
    private UserInfoMapper userInfoMapper;

    @Override
    public void add(UserInfo userInfo) {
   
        userInfoMapper.add(userInfo);
    }

    @Override
    public void delete(Integer id) {
   
        userInfoMapper.delete(id);
    }

    @Override
    public void update(UserInfo userInfo) {
   
        userInfoMapper.update(userInfo);
    }

    @Override
    public UserInfo queryById(Integer id) {
   
        return userInfoMapper.queryById(id);
    }

    @Override
    public List<UserInfo> queryAll() {
   
        return userInfoMapper.queryAll();
    }
}
```

#### 创建controller层

这里我先去pom中配置一个fastjson依赖这是阿里巴巴开源的，用来转换成JSON和类的格式的。

```java
<!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.78</version>
        </dependency>
```

我使用了RestFull风格去实现路径的请求


![img](img/faada5158df5da458cd1783a66b5fc19.png)

代码

```java
//@Controller 控制层需要的注解
//@RestController 使用这个也是可以的，但是使用后他里面所有请求返回的都是字符串！
//一般只需要作为接口放回JSON格式数据的话推荐使用@RestController
//@Controller这个是可以与Thymeleaf模板引擎使用时可以返回一个页面的
@Controller
//@RequestMapping指定路径名
//@RequestMapping("/test")用这个来指定路径也是可以的
@RequestMapping(value = "/test")
public class UserInfoController {
   
    //获取到UserInfoService
    @Autowired
    private UserInfoService userInfoService;

    //Get请求
    @GetMapping
    //@ResponseBody 注释后表示放回的是字符串
    @ResponseBody
    public String queryAll(){
   
        List<UserInfo> userInfoList = userInfoService.queryAll();
        return JSON.toJSONString(userInfoList);
    }

    //使用了RestFull风格
    @GetMapping("/{id}")
    @ResponseBody
    public String query(@PathVariable(value = "id")Integer id){
   
        UserInfo userInfo = userInfoService.queryById(id);
        List<UserInfo> userInfoList = new ArrayList<>();
        userInfoList.add(userInfo);
        return JSON.toJSONString(userInfoList);
    }

    //post请求
    //@RequestBody 表示接收请求是JSON格式的数据
    @PostMapping
    @ResponseBody
    public String add(@RequestBody UserInfo userInfo){
   
        userInfoService.add(userInfo);
        return "添加OK";
    }

    //Delete请求
    @DeleteMapping(value = "/{id}")
    @ResponseBody
    public String delete(@PathVariable("id")Integer id){
   
        userInfoService.delete(id);
        return "删除成功";
    }

    //Put请求
    @PutMapping("/{id}")
    @ResponseBody
    public String update(@PathVariable("id")Integer id,
            @RequestBody UserInfo userInfo){
   
        userInfo.setId(id);
        userInfoService.update(userInfo);
        return "修改成功";
    }
}
```

### 第五步测试请求

本人测试使用的工具是Postman Postman下载路径： [https://app.getpostman.com/app/download/win64](https://app.getpostman.com/app/download/win64)

查询测试


![img](img/9d9a64498493f1ed2fac02d11127926b.png)


![img](img/538ca1f352fae99e4112a05d659e3641.png)

查询没问题

增加数据测试


![img](img/65ca9281dad6b9221821c921156d4543.png)

此时数据库数据也多了一条数据


![img](img/399fa9c02b5a0dd29f9f733f5a405b34.png)

修改测试


![img](img/86bf2325751f80ed5b42741aa2015b2a.png)

此时数据库的数据也发生了改变


![img](img/d25c22fc93ee4d14a32e0567aa34f3eb.png)

删除测试


![img](img/b1675130d9cd6a126de3eed641c2168d.png)

此时数据就被删除了


![img](img/ec81ca4b9547c8a4a4e402149ff54ca3.png)

**源码路径：** [https://gitee.com/mehao123/meTest](https://gitee.com/mehao123/meTest)

