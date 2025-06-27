# Java 面向对象编程三大特性讲解：封装、继承、多态

[TOC]



## 一、封装（Encapsulation）

### ✅ 概念
封装是一种把对象的状态（属性）和行为（方法）捆绑在一起，并隐藏内部实现细节，仅通过对外公开的接口来访问对象的机制。

### ✅ 为什么要封装？
- **安全性**：防止外部直接访问或修改内部数据。
- **维护性**：更容易修改内部实现而不影响外部代码。
- **模块化**：每个对象是一个独立的模块。

### ✅ 关键实现方式
- 使用 `private` 关键字隐藏属性。
- 提供 `public` 的 `getter/setter` 方法访问私有属性。

### ✅ 示例代码
```java
public class Person {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        if(name != null && !name.isEmpty()){
            this.name = name;
        }
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if(age >= 0){
            this.age = age;
        }
    }
}
```





## 二、继承（Inheritance）

### ✅ 概念

继承是面向对象编程中让一个类（子类）获得另一个类（父类）成员变量和方法的机制。

Java 使用 `extends` 关键字实现类的继承，一个子类只能继承一个父类（单继承）。

### ✅ 优点

- **代码复用**：子类无需重复编写父类中已有的方法。
- **结构清晰**：通过父类抽象定义公共行为。
- **增强功能**：子类可以拓展父类功能。

### ✅ 关键词

- `extends`：用于声明继承关系。
- `super`：访问父类的属性、方法或构造函数。

### ✅ 示例代码

```java
// 父类
public class Animal {
    public void eat() {
        System.out.println("动物在吃东西");
    }
}

// 子类
public class Dog extends Animal {
    public void bark() {
        System.out.println("狗在叫");
    }
}
```

### ✅ 使用 super 关键字

```java
public class Dog extends Animal {
    public void eat() {
        System.out.println("狗先闻一闻...");
        super.eat(); // 调用父类方法
    }
}
```

### ✅ 构造函数中的 super

```java
public class Animal {
    public Animal(String name) {
        System.out.println("动物名字是：" + name);
    }
}

public class Dog extends Animal {
    public Dog() {
        super("小黑"); // 调用父类构造器
    }
}
```

------

## 三、多态（Polymorphism）

### ✅ 概念

多态指的是：同一个方法调用，在不同对象上会表现出不同的行为。

### ✅ 多态三要素（面试必问）

1. **继承**：子类继承父类。
2. **重写**：子类重写父类方法。
3. **父类引用指向子类对象**。

### ✅ 实现方式

- 方法重写（Override）
- 接口实现
- 向上转型 + 动态绑定

### ✅ 示例代码：方法重写 + 多态调用

```java
class Animal {
    public void makeSound() {
        System.out.println("动物发出声音");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("喵喵喵");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("汪汪汪");
    }
}

public class Test {
    public static void main(String[] args) {
        Animal a1 = new Cat();
        Animal a2 = new Dog();

        a1.makeSound(); // 喵喵喵
        a2.makeSound(); // 汪汪汪
    }
}
```

### ✅ 向上转型（Upcasting）

```java
Animal animal = new Dog(); // 父类引用指向子类对象
```

### ✅ 动态绑定

Java 在运行时会根据对象的实际类型来决定调用哪个方法，而不是编译时决定，这就是动态绑定。

### ✅ 多态的好处

- 接口统一，调用灵活
- 扩展方便：新增类只需实现接口或继承父类
- 解耦合，适合架构设计

### ✅ 注意事项

- 不能通过父类引用访问子类特有方法，除非强制类型转换。
- 多态只作用于成员方法，**成员变量不具备多态性**。

```java
class A {
    public int x = 1;
    public void print() {
        System.out.println("A");
    }
}

class B extends A {
    public int x = 2;
    public void print() {
        System.out.println("B");
    }
}

public class Test {
    public static void main(String[] args) {
        A a = new B();
        System.out.println(a.x); // 输出1，而不是2
        a.print();               // 输出B，多态体现
    }
}
```

------

### ✅为什么要“父类引用指向子类对象”？

1. **实现多态的关键**

   - 只有这样，调用的方法才可能在运行时根据实际对象类型“动态绑定”，输出不同结果（这就是多态的本质）。
   - 如果用子类引用指向子类对象，只能调用子类的方法，**体现不出多态**的威力。

2. **举例理解**

   ```java
   Animal a = new Dog(); // 父类引用指向子类对象
   a.shout(); // 调用的是Dog重写后的shout方法
   ```

   - 如果这样写：`Dog d = new Dog(); d.shout();`，只能调用Dog的功能，失去多态意义。

------

### ✅好处（为什么设计成这样）

1. **统一接口，灵活扩展**
   - 用父类类型定义变量，可以用同一种方式处理不同子类对象。
   - 例如：`Animal[] animals = {new Dog(), new Cat(), new Pig()};`
      遍历 animals，统一调用 `shout()`，各自表现不同。
2. **解耦合，易于维护和拓展**
   - 新增子类无需改动已有逻辑，只要重写父类方法即可。
   - 只关心“**能做什么**”（方法），不关心“**具体怎么做**”（子类实现）。
3. **代码复用与架构设计**
   - 便于设计出高可扩展性的框架（如工厂模式、策略模式等，都是靠这种引用实现的）。









## 四、小结对比表格

| 特性 | 核心含义           | 关键词                 | 好处                   |
| ---- | ------------------ | ---------------------- | ---------------------- |
| 封装 | 隐藏细节，保护数据 | private, getter/setter | 安全性、模块化、可维护 |
| 继承 | 子类复用父类内容   | extends, super         | 复用性、层次结构       |
| 多态 | 同一操作不同表现   | override, upcasting    | 扩展性、接口灵活调用   |

