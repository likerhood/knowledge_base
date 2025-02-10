# jvm动态代理作用理解

### 1. **什么是动态代理？**

动态代理的核心概念是：**在程序运行时，自动生成一个代理类来代替目标类，进行一些额外的操作或增强功能**。

你可以把它想象成一种“中介”或“替身”，当你调用方法时，代理类会拦截并执行一些额外的操作，然后再将请求转发给真正的实现类。

### 2. **简单的例子：打车服务**

假设你想通过一个打车应用叫车，打车公司有很多司机（实现了相同的接口），你只需要告诉应用“我要打车”，然后应用根据你的需求给你一个司机。**这个应用的设计就可以用代理模式来实现**。

#### 角色说明：

- **真实角色（司机）**：实现了 `Driver` 接口，提供具体的服务。
- **代理角色（打车应用）**：接收用户的请求（例如：叫车），可以在用户调用司机方法前后做一些处理（比如：计费、统计等）。

### 3. **代码实现：**

首先，我们定义一个 `Driver` 接口，它包含一个 `drive()` 方法。

```java
// 司机接口
public interface Driver {
    void drive(); // 司机开车
}
```

接着，我们有一个具体的 `RealDriver` 类，它实现了 `Driver` 接口。

```java
// 具体司机类
public class RealDriver implements Driver {
    @Override
    public void drive() {
        System.out.println("司机正在开车...");
    }
}
```

### 4. **代理角色：**

我们希望创建一个代理来代替 `RealDriver`，通过代理在方法调用之前和之后做一些额外的处理，比如记录日志、收费等。

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

// 代理类：通过 JDK 动态代理创建
public class DriverProxy implements InvocationHandler {

    private Object realDriver;

    // 构造方法，传入真实的司机对象
    public DriverProxy(Object realDriver) {
        this.realDriver = realDriver;
    }

    // 重写 invoke 方法：当调用代理对象的方法时，会进入这个方法
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 方法调用前的额外逻辑：比如记录日志
        System.out.println("代理：开始计费...");
        
        // 调用真实司机的方法
        Object result = method.invoke(realDriver, args);
        
        // 方法调用后的额外逻辑：比如结束计费
        System.out.println("代理：结束计费...");
        
        return result;
    }

    // 创建代理对象
    public static Driver createProxy(Object realDriver) {
        return (Driver) Proxy.newProxyInstance(
                realDriver.getClass().getClassLoader(), // 类加载器
                new Class[]{Driver.class}, // 实现的接口
                new DriverProxy(realDriver) // 代理对象
        );
    }
}
```

### 5. **如何使用代理？**

现在，我们可以用代理来代替 `RealDriver`。当我们调用代理对象的方法时，实际是通过代理来拦截并增强方法的行为。

```java
public class Main {
    public static void main(String[] args) {
        // 创建真实司机对象
        Driver realDriver = new RealDriver();
        
        // 创建代理对象
        Driver proxyDriver = DriverProxy.createProxy(realDriver);
        
        // 调用代理对象的方法
        proxyDriver.drive();  // 这将触发代理中的额外逻辑
    }
}
```

### 6. **输出结果：**

```
代理：开始计费...
司机正在开车...
代理：结束计费...
```

### 7. **这个代理做了什么？**

- 代理对象拦截了调用 `drive()` 方法。
- 代理对象在调用 `realDriver.drive()` 方法前后，分别添加了计费的逻辑。
- 代理类不需要了解 `RealDriver` 的具体实现，它只依赖于 `Driver` 接口。

### 8. **为什么要使用动态代理？**

- **解耦**：客户端（应用）只关心接口，而不关心具体的实现。这样就可以将行为（例如计费、日志等）与核心功能（司机的开车）解耦。
- **增强功能**：通过代理，我们可以在不修改 `RealDriver` 类的情况下，增加一些额外的功能（比如计费、日志等）。如果不使用代理，每次都需要修改 `RealDriver` 类，这不利于扩展和维护。
- **灵活性**：通过动态代理，我们可以在运行时决定使用哪个代理对象，这样非常灵活，适用于需要在运行时动态决定行为的场景。

### 9. **总结**

- **动态代理** 是通过代理对象在运行时拦截方法调用，并在方法前后做一些额外的操作（比如日志、性能监控、缓存等）。
- **好处**：提高了代码的复用性、扩展性和灵活性，同时减少了代码的耦合度。
- **应用场景**：日志记录、性能监控、事务管理、缓存等。



