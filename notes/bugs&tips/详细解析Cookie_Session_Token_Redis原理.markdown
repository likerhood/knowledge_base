# 详细解析Cookie_Session_Token_Redis原理

[toc]



## 1. Cookie和Session

### 1.1 Http是无状态的，有会话的

浏览器访问服务器时，浏览器和服务器之间就建立了连接，连接后浏览器可以向服务器发送多次请求，并且这多次请求之间是没有任何关系的，从服务器的角度而言，即便同一个浏览器像我发送了多次请求，我也不认为这些请求之间有什么关系，仍然把每次请求当做陌生人俩对待。他不对之前的请求和响应状态进行管理，无法根据之前的状态对本次请求进行处理。

### 1.2 Cookie会话机制

Cookie是由服务器创建发送到用户浏览器并保存在浏览器端的一小块数据，当浏览器再次向同一个服务器发送请求时救护携带上这块数据并发送给服务器。如果不设置Cookie的生存时间，Cookie就保存在浏览器内存中，当浏览器窗口关闭时，Cookie就会失效。如果设置了Cookie的生存时间，Cookie就会保存在硬盘上，知道生存时间过了Cookie才会过期。Cookie数据是以key=value形式存储的。


![img](img/8df1f8ba7878807fbcb19b70155e4e89.png)

Cookie会话管理机制：

（1）浏览器端第一次发送请求到服务器端 （2）服务器端创建Cookie，该Cookie中包含用户的信息，然后在响应时在响应头中将该Cookie发送到浏览器端 （3）将Cookie保存在浏览器端，当再次访问服务器端时会在请求头中携带服务器端创建的Cookie （4）服务器端通过Cookie中携带的数据区分不同的用户

```java
@RequestMapping(path = "/cookie/set", method = RequestMethod.GET)
@ResponseBody
public String setCookie(HttpServletResponse response) {
   
    // 创建cookie 其中key:code,value:随机字符串
    Cookie cookie = new Cookie("code", CommunityUtil.generateUUID());
    // 设置cookie生效的范围：即哪些请求会携带Cookie
    cookie.setPath("/community/alpha");
    // 设置cookie的生存时间,单位s，这样Cookie就会存在硬盘中，而不是内存中，只要过了生存时间才会失效
    cookie.setMaxAge(60 * 10);
    // 发送cookie，将cookie响应给浏览器
    response.addCookie(cookie);

    return "set cookie";
}

//获取cookie，从请求头中获取
@RequestMapping(path = "/cookie/get", method = RequestMethod.GET)
@ResponseBody
public String getCookie(@CookieValue("code") String code) {
   
    System.out.println(code);
    return "get cookie";
}
```


![img](img/f562837718e21748ad3e012918dee48f.png)

将敏感的用户信息存放在cookie中是很不安全的，造成cookie截获和cookie欺骗，而且cookie可以存放的内部才能很小，不能存放太多数据。一般对于比较敏感的数据不会选择存放在cookie中，而是存放在session中。

### 1.3 Session会话机制

Session可以将数据存放在服务端，而不是客户端，因此相对而言比较安全。一般可以将用户登录的用户名和密码存放在Session中。


![img](img/04d0260ea2faf5f32572a7e3ea28f035.png)

① 浏览器访问服务器，服务器就会创建一个Session对象，保存在服务器端，浏览器和Session之间的对应关系依赖于Cookie。

② 响应数据时，服务器底层自动创建一个Cookie，通过这个Cookie携带sessionId，这个sessionId就是这个对象的唯一标识，然后将Cookie保存在浏览器端

③ 再次访问服务器时就会发送给服务器，服务器得到sessionId后就可以去服务器端找对应的session了

Session是存在服务器的内存中， 每个会话对应一个sessionId，在响应时服务器会自动创建一个Cookie，JSESISONID作为key，在服务端创建的Session对象的sessionId作为value， 响应给浏览器端。这个SessionID就是一个唯一表示，用来区分session对象。

```java
/**
* session中可以存放任何数据，但是cookie中只能存放字符串，
* 因为Cookie需要来回传，不能存放大量数据，而且客户端其他类型数据也不能识别，但是字符串可以识别
*
* 向session中存放两条数据
*/
@RequestMapping(path = "/session/set", method = RequestMethod.GET)
@ResponseBody
public String setSession(HttpSession session) {
   
    session.setAttribute("id", 1);
    session.setAttribute("name", "Test");
    return "set session";
}
```

可以看到session对象有一个sessionId，这个sessionId通过Cookie携带存放在客户端，而用户信息存放在服务端


![img](img/b30bb90345ada69e588231dfd9cc77cb.png)

从Session中取值：

```java
@RequestMapping(path = "/session/get", method = RequestMethod.GET)
@ResponseBody
public String getSession(HttpSession session) {
   
    System.out.println(session.getAttribute("id"));
    System.out.println(session.getAttribute("name"));
    return "get session";
}
```


![img](img/d8131a451148929f58655037b08fe3d7.png)

### 1.4 Cookie和Session的区别？

（1）数据存放位置不同：Cookie存放在浏览器上，session存放在服务器上

（2）安全程度不同：cookie并不安全，别人可以解析本地的cookie进行cookie欺骗，对于登录等重要的信息可以存放在session中，而对于不重要的信息可以存放在cookie中。

（3）性能使用程度不同：session会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能,考虑到减轻服务器性能方面，应当使用cookie。

（4）数据存储大小不同：单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie，而session则存储与服务端，浏览器对其没有限制。

## 2. 为什么要用token验证，不用session?

### 2.1 Session会话

session表示会话，会话是指一个浏览器和一个服务器进行通信的过程。session存储于服务器，可以理解为一个状态列表，拥有一个唯一识别符号sessionId。

session的使用方式：客户端的cookie存放session_id，服务端的session保存用户信息，浏览器再次访问服务器时根据session_id到session中查找用户信息。

具体流程为：

① 当用户第一次通过浏览器使用用户名和密码访问服务器请求登录时，服务器会验证用户信息

② 登录成功后，会将用户信息保存在服务器端的session中，将session_id通过cookie响应给浏览器

③ 当用户再次登录时，会携带session_id，服务器拿着session_id找到session对象，查询用户信息，查询出后将用户信息返回给浏览器，从而使用户保持登录状态。


![img](img/8e9c8ac189d464d021820b983ffdb5e4.png)

session像是这个用户登录了应用，用户把全部信息存放在此应用，应用拥有完整的用户信息。

session：注册登录-&gt;服务端将user存入session-&gt;将sessionid存入浏览器的cookie-&gt;再次访问时根据cookie里的sessionid找到session里的user。

### 2.2 Session会话的弊端

（1）服务器压力增大

通常session是存储在内存中的，每个用户通过认证之后都会将session数据保存在服务器的内存中，而当用户量增大时，服务器的压力增大。

（2）CSRF跨站伪造请求攻击

session是基于cookie进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。

（3）无法实现session共享。

如果将来搭建了多个服务器，虽然每个服务器都执行的是同样的业务逻辑，但是session数据是保存在内存中的（不是共享的），用户第一次访问的是服务器1，当用户再次请求时可能访问的是另外一台服务器2，服务器2获取不到session信息，就判定用户没有登陆过。

### 2.3 token令牌机制

token一般翻译成令牌，一般是用于验证用户身份的数据，可以用url传参，也可以用post提交，也可以夹在http的header中。它相当于session中的session_id，是一个字符串，存放在浏览器中。

注意：session数据默认存放在服务器的内存中，因此当用户量变多时，服务器压力就会增大。如果将session数据存放在数据库中或者redis缓存中，就要建立session_id相关的数据库表，把session数据通过session_id存放在数据库中比较复杂，不如直接使用token代替session_id，将session数据存放在数据库中。

token的验证方式：由于服务器内存中并没有存储token数据，因此需要先从数据库中查询当前token，服务器再验证否有效；

具体流程为：

① 当浏览器第一次通过用户名和密码访问服务器时，服务器收到请求后会去验证用户信息。

② 登录成功后，服务端会签发一个token，再将这个token响应给浏览器，同时将token和对应的用户信息保存在数据库或缓存中

③ 浏览器收到token后，把它存储在Cookie或者localStorage中，以后每次向服务端请求资源的时候需要带着服务端签发的 token

④ 服务器收到请求后，会从数据库或者缓存中查询对应的token，如果验证成功，就向浏览器返回请求的数据，保持登录的状态。


![img](img/52383a19033bf6951668eaac0d249484.png)

这种方式也是我在项目中使用的方式，我并没有使用session存放用户信息，一是因为session数据默认存放在服务器内存中，为了减小服务器的压力，二是因为session需要依赖于Cookie，会造成CSRF（跨站请求伪造）的风险，三是因为用户信息存放在服务器内存中，为了避免session不能共享的问题。因此考虑到这些问题，我在项目中一开始将token和用户信息存放在了mysql数据库中，每次用户请求登录时携带token，通过token去数据库中认证，如果认证成功，那么就将请求数据返回给浏览器实现登录状态的保持。后来为了提高性能，我又对项目做了优化，将token和用户信息存在了redis缓存中，避免频繁访问数据库。因此方用户请求登录时，通过token去redis缓存中查询token实现认证。

token的优势：①无状态、可扩展 ②支持移动设备 ③跨程序调用 ④安全

### 2.4 token如何出现的？

token其实借鉴了cookie和session的工作原理，解决session依赖于单个服务器不能实现session共享的问题：单体应用时用户信息保存在session中，不会出现问题，但是如果有多太服务器就会出现问题。比如用户在A服务器登录后，session就存在A服务器中，但是之后第二次请求分到了B服务器，由于B服务器没有用户的session数据，因此用户还要重新登录。

① 在session会话管理中，cookie中保存的是session_id，这是一个具有唯一性标识的字符串，因为我们也可以使用一个具有唯一性标识的字符串返回给浏览器，保存在浏览器内存中。

② 服务器端存放的是session数据，每个session对象包括session_id和session数据中的键值对，这不就是redis中的哈希表吗？因此可以使用redis的哈希类型来模拟服务器的session。

③ 因此我们可以在用户第一次请求是生成一个全局唯一的token返回给浏览器，同时将用户信息保存在redis中并设置过期时间（session的过期时间为30分钟），之后浏览器的每次请求都带着这个token，服务器根据每次请求到redis中查找对应的用户信息。

### 2.5 JWT（Json web token）

jwt的由三部分组成，官网：https://jwt.io/

```java
头部（Header）
    用于描述关于该JWT的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个JSON对象。
    然后将其进行base64编码，得到第一部分
{
   
"typ": "JWT",
"alg": "HS256"
}

载荷（Payload）
    一般添加用户的相关信息或其他业务需要的必要信息。但不建议添加敏感信息，因为该部分在客户端可解密
    （base64是对称解密的，意味着该部分信息可以归类为明文信息）
    然后将其进行base64编码，得到第二部分

{
    "iss": "JWT Builder", 
  "iat": 1416797419, 
  "exp": 1448333419, 
  "aud": "www.example.com", 
  "sub": "aaa@example.com", 
  "Email": "aaa@example.com", 
  "Role": [ "admin", "user" ] 
}
iss:该JWT的签发者，是否使用是可选的；
sub:该JWT所面向的用户，是否使用是可选的；
aud:接收该JWT的一方，是否使用是可选的；
exp(expires):什么时候过期，这里是一个Unix时间戳，是否使用是可选的；
iat(issued at):在什么时候签发的(UNIX时间)，是否使用是可选的；
nbf (Not Before):如果当前时间在nbf里的时间之前，则Token不被接受；是否使用是可选的；
jti:JWT的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。

签名（Signature）
    需要base64加密后的header和base64加密后的payload使用"."连接组成的字符串，
    然后通过header中声明的加密方式进行加盐secret组合加密（在加密的时候，我们还需要提供一个密钥（secret），加盐secret组合加密）
    然后就构成了jwt的第三部分。
最后，将这一部分签名也拼接在被签名的字符串后面，我们就得到了完整的JWT
```

jwt认证方式：注册登录-&gt;服务端将生成一个token，并将token与user加密生成一个密文-&gt;将token+user+密文数据 返回给浏览器-&gt;再次访问时传递token+user+密文数据，后台会再次使用token+user生成新密文，与传递过来的密文比较，一致则正确。

具体流程：

①认证成功后，会对当前用户数据进行加密，生成一个加密字符串token，返还给客户端（服务器端并不进行保存）

②浏览器会将接收到的token值存储在Local Storage中，（通过js代码写入Local Storage，通过js获取，并不会像cookie一样自动携带）


③再次访问时服务器端对token值的处理：服务器对浏览器传来的token值进行解密，解密完成后进行用户数据的查询，如果查询成功，则通过认证，实现状态保持，所以，即时有了多台服务器，服务器也只是做了token的解密和用户数据的查询，它不需要在服务端去保留用户的认证信息或者会话信息，这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利，解决了session无法共享的弊端。 ![img](img/a17baef3eb8ae83a3b645e59ccbe3443.png)

## 3. 存放验证码

### 3.1 将验证码存放在Session中

```java
@RequestMapping(path = "/kaptcha", method = RequestMethod.GET)
public void getKaptcha(HttpServletResponse response, HttpSession session) {
   
    // 生成验证码
    String text = kaptchaProducer.createText();
    BufferedImage image = kaptchaProducer.createImage(text);

    // 将验证码存入session
    session.setAttribute("kaptcha", text);

    // 将突图片输出给浏览器
    response.setContentType("image/png");
    try {
   
        OutputStream os = response.getOutputStream();
        ImageIO.write(image, "png", os);
    } catch (IOException e) {
   
        logger.error("响应验证码失败:" + e.getMessage());
    }
}
```


![img](img/861d1ae50dc7b33f64b495dbcecd8253.png)

将生成的验证码放在session中，当响应时就会自动生成一个Cookie，Cookie中携带了这个session对象的sessiod_id，并保存在浏览器中，当登录时就会在请求头中通过cookie携带session_id，通过session_id找到对应的session对象，进而取出对应的验证码。

```java
@RequestMapping(path = "/login", method = RequestMethod.POST)
public String login(String username, String password, String code, boolean rememberme, Model model, HttpSession session, HttpServletResponse response) {
   

    // 检查验证码，用户生成的验证码一开始是是放在session中的，需要通过session将验证码取出来
    String kaptcha = (String) session.getAttribute("kaptcha");

    if (StringUtils.isBlank(kaptcha) || StringUtils.isBlank(code) || !kaptcha.equalsIgnoreCase(code)) {
   
        model.addAttribute("codeMsg", "验证码不正确!");
        return "/site/login";
    }
}
```

### 3.2 将验证码存放在redis中

使用redis存放验证码，提高性能：

① 验证码需要频繁的访问和刷新，对性能要求较高。

② 验证码不会永久保存，通常在很短的时间内就会失效

③ 验证码存在session中，分布式部署时存在session共享的问题

```java
//构建redis的key，key中存放的用户的唯一标识owner
public class RedisKeyUtil {
   
    private static final String SPLIT = ":";
    private static final String PREFIX_KAPTCHA = "kaptcha";

    // 登录验证码
    public static String getKaptchaKey(String owner) {
   
        //验证码和用户是相关的，需要识别用户，但是又不可能和userId绑定到一起，因为此时用户还没登录
        //在用户访问登录页面的时候，给他发一个凭证owner，让他存在Cookie中，我们用这个字符串owner来临时标识这个用户
        return PREFIX_KAPTCHA + SPLIT + owner;
    }
}

@Controller
public class LoginController implements CommunityConstant {
   
	@Autowired
    private UserService userService;

    @Autowired
    private Producer kaptchaProducer;

    @Value("${server.servlet.context-path}")
    private String contextPath;

    @Autowired
    private RedisTemplate redisTemplate;

    @RequestMapping(path = "/kaptcha", method = RequestMethod.GET)
    public void getKaptcha(HttpServletResponse response/*, HttpSession session*/) {
   
        // 生成验证码
        String text = kaptchaProducer.createText();
        BufferedImage image = kaptchaProducer.createImage(text);

        // 验证码的归属，用来将验证码和用户关联起来的owner
        String kaptchaOwner = CommunityUtil.generateUUID();
        //这个owner需要存放在Cookie中发送给客户端
        Cookie cookie = new Cookie("kaptchaOwner", kaptchaOwner);
        cookie.setMaxAge(60);
        cookie.setPath(contextPath);
        response.addCookie(cookie);

        // 将验证码存入Redis中，redis的key存放的是验证码的owner，value存放的是owner对应的验证码
        String redisKey = RedisKeyUtil.getKaptchaKey(kaptchaOwner);
        redisTemplate.opsForValue().set(redisKey, text, 60, TimeUnit.SECONDS);

        // 将图片输出给浏览器
        response.setContentType("image/png");
        try {
   
            OutputStream os = response.getOutputStream();
            ImageIO.write(image, "png", os);
        } catch (IOException e) {
   
            logger.error("响应验证码失败:" + e.getMessage());
        }
    }

    @RequestMapping(path = "/login", method = RequestMethod.POST)
    public String login(String username, String password, String code, boolean rememberme, Model model,HttpServletResponse response) {
   
        // 登录的时候先从cookie中取出验证码的归属kaptchaOwner
        String kaptcha = null;
        if (StringUtils.isNotBlank(kaptchaOwner)) {
   
            //得到redis的key
            String redisKey = RedisKeyUtil.getKaptchaKey(kaptchaOwner);
            //通过redis的key得到验证码
            kaptcha = (String) redisTemplate.opsForValue().get(redisKey);
        }

        if (StringUtils.isBlank(kaptcha) || StringUtils.isBlank(code) || !kaptcha.equalsIgnoreCase(code)) {
   
            model.addAttribute("codeMsg", "验证码不正确!");
            return "/site/login";
        }
    }
}
```

总结：

将验证码存放在session中时，验证码就是和session_id关联起来的，当用户首次访问登录页面的时候，就会在后端生成一个验证码，并将session_id通过cookie响应给浏览器，当用户登录的时候就会在请求头中通过cookie携带这个session_id，通过session_id找到对应的session对象，从而获得验证码。用户和验证码是通过具有唯一性标识的session_id关联起来的，因此当我们向把验证码存放在redis中，也需要模仿这个原理和过程。

我们使用的owner这个唯一标识就相当于session_id，也可以把它当成token，总之就是一个全局唯一的字符串。当验证码生成后，服务器就生成一个owner(token)，然后通过Cookie将这个owner返回给浏览器，并在浏览器中保存下来，当用户登录的时候，就会通过cookie携带owner，然后通过owner找到对应的redis的key，进而验证验证码是否正确。这个过程其实就是token的验证过程。只不过换了一个名字而已。

## 4. 存放标识用户信息的token

### 4.1 将token存放在数据库中


![img](img/00afac4020b7583f144e6631eed61de1.png)

首先用户请求登录，服务器会进行验证，验证成功后会生成一个登录凭证LoginTicket对象，将登录凭证存入数据库中，同时将代表session_id的ticket通过Cookie返回给客户端，并存放在客户端。当下次登录时在请求头中携带ticket，服务器从数据库表中查询对应的ticket并验证是否正确，验证成功后就可以得到用户信息了。

```java
@Service
public class UserService implements CommunityConstant {
   

    @Autowired
    private UserMapper userMapper;
    @Autowired
    private LoginTicketMapper loginTicketMapper;
    
	//用户登录
    public Map<String, Object> login(String username, String password, int expiredSeconds){
   
        Map<String, Object> map = new HashMap<>();

        // 服务器端进行验证，验证成功后生成登录凭证.....

        // 生成登录凭证
        LoginTicket loginTicket = new LoginTicket();
        loginTicket.setUserId(user.getId());
        loginTicket.setTicket(CommunityUtil.generateUUID());
        //登录状态时status=0，代表凭证有效
        loginTicket.setStatus(0);
        //凭证的过期时间
        loginTicket.setExpired(new Date(System.currentTimeMillis() + expiredSeconds * 1000));
      	//将登录凭证和用户信息都存入数据库中
        loginTicketMapper.insertLoginTicket(loginTicket);

        map.put("ticket", loginTicket.getTicket());
        return map;
    }
    
    //退出登录
    public void logout(String ticket) {
   
        loginTicketMapper.updateStatus(ticket, 1);
    }
}

@Controller
public class LoginController implements CommunityConstant {
   
    @Autowired
    private UserService userService;

    @Value("${server.servlet.context-path}")
    private String contextPath;

    //用户登录
    @RequestMapping(path = "/login", method = RequestMethod.POST)
    public String login(String username, String password, String code, boolean rememberme,Model model, HttpServletResponse response) {
   

        // 检查账号,密码
        int expiredSeconds = rememberme ? REMEMBER_EXPIRED_SECONDS : DEFAULT_EXPIRED_SECONDS;
        Map<String, Object> map = userService.login(username, password, expiredSeconds);
        //如果map中包含了ticket，说明登录成功
        if (map.containsKey("ticket")) {
   
            //创建Cookie，将tiket存放在Cookie中返回给客户端
            Cookie cookie = new Cookie("ticket", map.get("ticket").toString());
            cookie.setPath(contextPath);
            cookie.setMaxAge(expiredSeconds);
            //将cookie响应给客户端
            response.addCookie(cookie);
            return "redirect:/index";
        } else {
   
            model.addAttribute("usernameMsg", map.get("usernameMsg"));
            model.addAttribute("passwordMsg", map.get("passwordMsg"));
            return "/site/login";
        }
    }
    
	//用户登出
    @RequestMapping(path = "/logout", method = RequestMethod.GET)
    public String logout(@CookieValue("ticket") String ticket) {
   
        userService.logout(ticket);
        return "redirect:/login";
    }
}
```

用户首次登录—》验证用户信息----》生成登录凭证----》将登录凭证和用户信息保存在数据库中—》创建Cookie对象，将ticket通过cookie响应给客户端。注意：这里只写了首次登录的逻辑，至于再次登录时通过凭证查询用户信息，这个逻辑需要用到拦截器，下次继续讨论。

### 4.2 将token存放在redis中

使用redis存储登录凭证，因此处理每次请求时，都需要查询用户的登录凭证，访问的频率较高，可以通过redis提高性能。因此将上面存放在数据库中的登录凭证改为存在redis中。

```java
//构建redis的key，因为需要通过ticket查询用户信息，因此将ticket作为redis的key
public class RedisKeyUtil {
   
    private static final String SPLIT = ":";
    private static final String PREFIX_TICKET = "ticket";
    // 登录的凭证
    public static String getTicketKey(String ticket) {
   
        return PREFIX_TICKET + SPLIT + ticket;
    }
}

@Service
public class UserService implements CommunityConstant {
   
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private RedisTemplate redisTemplate;

    public Map<String, Object> login(String username, String password, int expiredSeconds) {
   
        Map<String, Object> map = new HashMap<>();
        // 验证信息

        // 验证成功后，生成登录凭证
        LoginTicket loginTicket = new LoginTicket();
        loginTicket.setUserId(user.getId());
        loginTicket.setTicket(CommunityUtil.generateUUID());
        loginTicket.setStatus(0);
        loginTicket.setExpired(new Date(System.currentTimeMillis() + expiredSeconds * 1000));
        //构建redis的key
        String redisKey = RedisKeyUtil.getTicketKey(loginTicket.getTicket());
        //将ticket和用户信息存在redis缓存中，key=ticket,value = loginTicket
        redisTemplate.opsForValue().set(redisKey, loginTicket);

        map.put("ticket", loginTicket.getTicket());
        return map;
    }

    public void logout(String ticket) {
   
        //登出时构建redis的key
        String redisKey = RedisKeyUtil.getTicketKey(ticket);
        //从redis中查询出loginTicket
        LoginTicket loginTicket = (LoginTicket) redisTemplate.opsForValue().get(redisKey);
        //将用户登录状态设置为失效
        loginTicket.setStatus(1);
        //再将loginTicket存入redis缓存中
        redisTemplate.opsForValue().set(redisKey, loginTicket);
    }
}

@Controller
public class LoginController implements CommunityConstant {
   
    @Autowired
    private UserService userService;

    @Value("${server.servlet.context-path}")
    private String contextPath;

    @Autowired
    private RedisTemplate redisTemplate;
    
    //用户登录
    @RequestMapping(path = "/login", method = RequestMethod.POST)
    public String login(String username, String password, String code, boolean rememberme, Model model, HttpServletResponse response) {
   

        // 检查账号,密码
        int expiredSeconds = rememberme ? REMEMBER_EXPIRED_SECONDS : DEFAULT_EXPIRED_SECONDS;
        Map<String, Object> map = userService.login(username, password, expiredSeconds);
        if (map.containsKey("ticket")) {
   
            Cookie cookie = new Cookie("ticket", map.get("ticket").toString());
            cookie.setPath(contextPath);
            cookie.setMaxAge(expiredSeconds);
            response.addCookie(cookie);
            return "redirect:/index";
        } else {
   
            model.addAttribute("usernameMsg", map.get("usernameMsg"));
            model.addAttribute("passwordMsg", map.get("passwordMsg"));
            return "/site/login";
        }
    }
	
    //用户登出
    @RequestMapping(path = "/logout", method = RequestMethod.GET)
    public String logout(@CookieValue("ticket") String ticket) {
   
        userService.logout(ticket);
        return "redirect:/login";
    }
}
```

用户首次登录—》服务器验证—》验证成功后生成登录凭证存放在redis中，key = ticket,value=loginTicket----》创建Cookie对象，通过cookie将ticket返回给客户端并保存下来。

## 5. redis缓存用户信息

处理每次请求时都需要通过凭证查询用户信息，因此可以通过将用户信息放在redis缓存中。

使用redis缓存的步骤： ①优先从缓存中取值 ②取不到值时初始化缓存数据 ③数据变更时清除缓存数据

```java
// 构建redis的key，因为获取凭证后查询用户信息是通过userId插询的，因此将userId设置为key
public static String getUserKey(int userId) {
   
    return PREFIX_USER + SPLIT + userId;
}

@Service
public class UserService implements CommunityConstant {
   
    
    // 1.优先从缓存中取值：查询用户时尝试从缓存中取值，先不去访问mysql
    private User getCache(int userId) {
   
        //构建redis的key=userId
        String redisKey = RedisKeyUtil.getUserKey(userId);
        //通过userId从缓存中取出用户信息User
        return (User) redisTemplate.opsForValue().get(redisKey);
    }

    // 2.取不到时初始化缓存数据：取不到用户信息时说明缓存中没有初始化，初始化
    private User initCache(int userId) {
   
        //数据来源于mysql
        User user = userMapper.selectById(userId);
        //构建redis的key
        String redisKey = RedisKeyUtil.getUserKey(userId);
        //向redis中存入用户信息，key= userId,value=user
        redisTemplate.opsForValue().set(redisKey, user, 3600, TimeUnit.SECONDS);
        return user;
    }

    // 3.数据变更时清除缓存数据：用户数据发生变化时，清除缓存
    private void clearCache(int userId) {
   
        String redisKey = RedisKeyUtil.getUserKey(userId);
        redisTemplate.delete(redisKey);
    }

    //每次获取登录凭证后，都要调用这个方法，因此访问非常频繁
    public User findUserById(int id) {
   
		//先从缓存中查询用户信息
        User user = getCache(id);
        if (user == null) {
   
            //查不到的时候，就会初始化redis缓存
            user = initCache(id);
        }
        return user;
    }

    public int activation(int userId, String code) {
   
        User user = userMapper.selectById(userId);
        if (user.getStatus() == 1) {
   
            return ACTIVATION_REPEAT;
        } else if (user.getActivationCode().equals(code)) {
   
            userMapper.updateStatus(userId, 1);
            //激活的时候对用户状态进行了修改，因此需要清除缓存
            clearCache(userId);
            return ACTIVATION_SUCCESS;
        } else {
   
            return ACTIVATION_FAILURE;
        }
    }

    public int updateHeader(int userId, String headerUrl) {
   
        int rows = userMapper.updateHeader(userId, headerUrl);
        //修改用户信息，清除缓存
        clearCache(userId);
        return rows;
    }
}
```

