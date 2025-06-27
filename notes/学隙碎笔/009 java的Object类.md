# Object 类知识点整理

## 一、Object类概念

- `Object` 是 Java 所有类的**根类**（父类）。
- Java 中每个类都直接或间接继承自 `Object` 类。
- `Object` 类定义了一组最基础、最通用的方法，为所有 Java 对象提供了公共行为。

------

## 二、Object类常用方法

### 1. `equals` 方法

#### 1.1 == 和 equals() 的区别

- 对**基本数据类型**：`==` 比较的是值。
- 对**引用数据类型**：`==` 比较的是对象的内存地址。
- Java 只有值传递，所以变量本质存储的都是值，引用类型变量存的是对象地址。

#### 1.2 equals() 方法适用场景

- **equals()** 只能用来判断两个对象是否相等，不能用于基本数据类型。
- **equals() 方法在 Object 类中定义**，所有类都继承了这个方法。

#### 1.3 Object 类的 equals() 实现

- Object 默认实现也是比较引用地址（和==等价）：

  ```java
  public boolean equals(Object obj) {
      return (this == obj);
  }
  ```

#### 1.4 equals() 方法的重写

- 如果**没有重写** equals()，默认比较地址。
- 如果**重写了** equals()（如 String、Integer等），则比较的是内容。
- 自定义类一般都需要重写 equals 方法，按业务需求比较内容属性。

**代码示例**

```java
String a = new String("ab");
String b = new String("ab");
String aa = "ab";
String bb = "ab";
System.out.println(aa == bb);         // true（同常量池对象）
System.out.println(a == b);           // false（不同引用）
System.out.println(a.equals(b));      // true（内容相等）
System.out.println(42 == 42.0);       // true（值类型自动转double比较）
```

- String 的 equals 方法重写后，比较字符串的内容值。

#### 1.6 String类的 equals() 实现简要分析

- 先判断引用是否相同
- 再判断类型是否为 String
- 逐字符比较 value 数组内容



------

### 2. `hashCode` 方法

- **作用**：返回对象的哈希码（int 整数）。
- **默认实现**：根据对象的内存地址生成。
- **重写建议**：若重写 `equals`，**必须重写 `hashCode`**，保证相等的对象哈希码也相等。
- **应用场景**：用于哈希容器，如 `HashMap`、`HashSet`。

#### 2.1 hashCode 方法的定义与本质

- `hashCode()` 是 Java `Object` 类中的一个 **实例方法**，返回对象的哈希码（int 型）。
- 作用：为对象生成一个“整数标识”，这个值用于支持基于哈希表的集合（如 `HashMap`、`HashSet`）的高效存取。
- 在 Java 中，**所有类**都直接或间接拥有 `hashCode()` 方法。

```java
public native int hashCode();
```

- 这是一个本地方法，不同的 JVM 实现细节略有不同，通常和对象内存地址、JVM对象元数据或随机数有关。

------

#### 2.2 hashCode 的用途及意义

- **主要用途**：快速定位对象在哈希表（如 HashMap/HashSet）中的存储位置。
- **作用场景**：只有涉及哈希表（“散列表”）时，`hashCode()` 才有实际作用，其他场景（如普通对象比较）基本无用。
- 哈希表底层依靠“数组”，对象通过 `hashCode()` 计算得到“索引”，再配合 `equals()` 判断对象内容。

**图解说明：**

```
"Hello"对象 --hashCode()--> 69609650  --->  散列表对应索引
```

------

#### 2.3 为什么要有 hashCode？（以 HashSet 为例）

- 向 `HashSet` 添加对象时，先计算对象的 hashCode，找到对应存储桶。
- 如果该存储桶已有对象，继续调用 `equals()` 逐个判断是否与新对象内容相等。
- 若内容也相等，则视为重复，不会加入。
- **优势**：极大提高查找和去重效率，只需在哈希冲突时才需要 equals 全量比对。

------

#### 2.4 hashCode 与 equals 的约定关系

- **强约定**：
  1. 如果 `a.equals(b)` 为 `true`，那么 `a.hashCode() == b.hashCode()` 必须成立。
  2. 反之，`a.hashCode() == b.hashCode()` 并不能保证 `a.equals(b)` 为 `true`（可能哈希碰撞）。
- **为什么？**
  - 若只重写 `equals` 而不重写 `hashCode`，集合会误判“重复对象”为“新对象”，导致 HashSet/HashMap 出现查找失败或重复元素。

------

#### 2.5 常见问题与面试高频问答

- **为什么只重写 equals 不重写 hashCode 会出错？**
  - 因为哈希表只根据 hashCode 先定位桶，如果 hashCode 不同，即使 equals 为 true，也不会被判定为同一个对象。
- **为什么有的对象 hashCode 相等却不 equals？**
  - 这叫“哈希冲突”。hashCode 计算方式有限，可能不同对象结果一样，但 equals 仍需判定内容是否真的一致。
- **为什么 JDK 不只用 hashCode？**
  - hashCode 相等只代表“可能相等”，真正的内容相等需要 equals 来最终判定。
- **总结口诀：**
  - hashCode 不同，一定不 equals
  - equals 为 true，hashCode 必须相同
  - hashCode 相同，未必 equals

------

#### 2.6 实现规范与开发建议

- **重写 equals 时，\**务必\**同时重写 hashCode**
- 推荐用 IDE 自动生成 hashCode/equals，保证一致性
- 在哈希集合类（如 HashSet、HashMap、Hashtable）中自定义类型用作 key 时，必须保证上述约定
- **hashCode 重写示例（常见写法）**：

```java
@Override
public int hashCode() {
    return Objects.hash(name, age); // 按内容属性生成hash值
}
```

------

#### 2.7 典型代码示例

```java
class Person {
    String name;
    int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
Person p1 = new Person("Tom", 18);
Person p2 = new Person("Tom", 18);
Set<Person> set = new HashSet<>();
set.add(p1);
set.add(p2); // 只会加入一个，因为equals和hashCode都相等
```

------

#### 2.8 小结

- `hashCode()` 提供对象的哈希码，常用于哈希表高效查找。
- `equals()` 判断对象内容是否相等，`hashCode()` 支持对象定位。
- 两者的正确配合，是保证 Java 集合（如 HashMap、HashSet）正常运行的关键。

------

### 3. `getClass` 方法

##### 3.1、Java 是“运行时类型识别”的语言，是什么意思？

- Java 是静态类型语言，变量类型在编译时就必须确定。
- 但是，为了灵活性，**Java 同时保留了对象在运行时的真实类型信息**，称为 **运行时类型识别（RTTI）**。
- 这种设计使得你在运行阶段也能识别对象的实际类型并进行操作。

🧠 本质理解：
 编译时编译器不知道 `Object obj` 实际是什么类型，但 JVM 在运行时能识别出 obj 是 `Student` 类的实例。

```java
Object obj = new Student();
System.out.println(obj.getClass().getName()); // 输出：Student
```

------

##### 3.2、Java 的执行过程回顾

1. Java 源码（.java） → 编译 → 字节码文件（.class）
2. JVM 运行字节码 → 创建对象、调用方法等
3. JVM 会为每个 `.class` 创建一个 **Class 对象**，用来表示类的元信息

🌟 这个 `Class` 对象，就是 Java 反射的核心。

------

##### 3.3、什么是反射机制？

- **反射是 Java 提供的一种在运行时操作类和对象的能力。**
- 借助 JVM 中的 `Class` 类，可以：
  - 加载类
  - 创建对象
  - 获取和修改字段（包括私有字段）
  - 获取并调用方法
  - 读取注解信息

------

##### 3.4、反射的触发入口 —— `getClass()`

- `getClass()` 是 `Object` 类中的方法，返回对象的运行时类。
- 它是最常用的反射入口之一。

```java
Person p = new Person();
Class<?> clazz = p.getClass();  // 获取 p 对象的类信息
```

------

##### 3.5、为什么要使用反射？它到底能做什么？

| 场景               | 说明                                                   |
| ------------------ | ------------------------------------------------------ |
| 配置控制加载类     | 配置文件中给出类名，用反射动态加载（Spring IoC）       |
| 框架中统一处理方法 | 根据注解自动调用方法（JUnit、Spring Boot）             |
| 工具类操作任意对象 | 比如：打印任意对象的字段、JSON 序列化（Gson、Jackson） |
| 依赖注入           | Spring 通过反射创建 Bean，并注入成员变量               |
| 动态代理           | AOP（面向切面编程）中，动态生成类并添加行为            |

------

##### 3.6、图解：Java 反射在框架中的作用



------

##### 3.7、反射代码简单示例（加载类 + 调用方法）

```java
// 类定义
public class Person {
    public void sayHello() {
        System.out.println("Hello!");
    }
}

// 调用代码
Class<?> clazz = Class.forName("Person"); // 加载类
Object obj = clazz.getDeclaredConstructor().newInstance(); // 创建实例
Method method = clazz.getMethod("sayHello"); // 获取方法
method.invoke(obj); // 调用方法
```

------

##### 3.8、反射的优缺点

| 优点           | 缺点                             |
| -------------- | -------------------------------- |
| 灵活、解耦     | 性能较差（通过 Method 调用较慢） |
| 可操作未知类型 | 代码复杂，难以调试               |
| 框架核心机制   | 破坏封装性，可能访问私有成员     |

------

##### 3.9、小结

- Java 虽然是静态语言，但通过 `Class` 和 `getClass()` 提供了运行时类型识别能力。
- 反射机制是 Spring、JUnit、MyBatis 等 Java 框架的核心支撑。
- 它让程序具备**“在运行时动态加载和操作类”**的能力，是面向框架设计和工具开发的利器。

------

### 4. `toString` 方法

- **作用**：返回对象的字符串表现形式。
- **默认实现**：类名 + “@” + 十六进制哈希码。
- **重写建议**：便于调试和日志，建议在自定义类中重写，返回有意义的信息。
-  `String` 类特殊，重写后返回自身内容，便于直接打印字符串。 

```java
@Override
public String toString() {
    return "Person{name='" + name + "', age=" + age + "}";
}
```

------

## 三、总结

| 方法名   | 作用                         | 重写建议/场景              |
| -------- | ---------------------------- | -------------------------- |
| equals   | 判断对象“内容”是否相等       | 自定义类需重写             |
| hashCode | 提供哈希码，用于哈希容器     | 重写equals必须重写hashCode |
| getClass | 获取对象运行时类信息         | 反射、调试                 |
| toString | 返回字符串表示，便于输出调试 | 建议重写，                 |







1. 静态类和内部类
2. 枚举和注解
3. 异常
4. 常用类
5. 集合
6. 泛型
7. 多线程
8. IO流
9. 网络编程
10. 反射
11. MySql
12. JDBC和数据库连接池
13. Spring
14. SpringBoot
15. 常用设计模式
16. 网盘项目
17. openai项目
18. 安卓搞一个项目
19. 编译器
20. 前端和前端框架
21. 算法和数据结构



