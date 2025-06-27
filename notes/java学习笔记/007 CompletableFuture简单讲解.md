# CompletableFuture 详解（小白友好版）

`CompletableFuture` 是 Java 8 引入的一个强大的异步编程工具

## 一、CompletableFuture 是什么？

### 1. 简单理解
想象你点了一份外卖：
- **同步方式**：站在门口一直等，直到外卖送到才能做其他事（阻塞）
- **异步方式**：下单后该干嘛干嘛，外卖到了会通知你（非阻塞）

`CompletableFuture` 就像那个外卖订单：
- 它是一个"未来会完成"的承诺
- 你可以在等待期间做其他事情
- 完成后会自动通知你

### 2. 基本概念
```java
CompletableFuture<String> future = new CompletableFuture<>();
```
- `CompletableFuture<String>` 表示这个"未来结果"是一个String类型
- 初始时结果还未计算完成
- 完成后可以通过它获取结果

## 二、为什么需要 CompletableFuture？

### 1. 同步 vs 异步
| 方式 | 特点                         | 类比                                   |
| ---- | ---------------------------- | -------------------------------------- |
| 同步 | 调用后必须等待结果返回       | 排队买奶茶，必须等到拿到奶茶才能离开   |
| 异步 | 调用后立即返回，结果后续处理 | 扫码点单后找座位等叫号，期间可以玩手机 |

### 2. 传统异步的问题
以前的Java异步编程需要：
- 手动创建线程
- 处理线程同步
- 自己管理回调
- 容易写出"回调地狱"

### 3. CompletableFuture 的优势
- 链式调用（避免回调地狱）
- 内置线程池管理
- 支持组合多个异步操作
- 异常处理机制完善

## 三、核心方法解析

### 1. 基本使用
```java
CompletableFuture<String> future = openAiSession.completions(request);

// 异步处理结果
future.thenAccept(result -> {
    System.out.println("收到结果: " + result);
});

// 异常处理
future.exceptionally(e -> {
    e.printStackTrace();
    return "默认结果";
});
```

### 2. 主要方法说明
| 方法              | 说明           | 示例                                         |
| ----------------- | -------------- | -------------------------------------------- |
| `thenAccept()`    | 处理正常结果   | `future.thenAccept(System.out::println)`     |
| `thenApply()`     | 转换结果       | `future.thenApply(String::toUpperCase)`      |
| `exceptionally()` | 处理异常       | `future.exceptionally(e -> "error")`         |
| `thenCompose()`   | 组合多个Future | `future.thenCompose(this::anotherAsyncCall)` |
| `allOf()`         | 等待所有完成   | `CompletableFuture.allOf(future1, future2)`  |

## 四、在 OpenAiSession 中的应用

### 1. 接口定义解读
```java
CompletableFuture<String> completions(ChatCompletionRequest request) throws Exception;
```
- **返回值**：`CompletableFuture<String>` 表示这是一个异步调用
- **String内容**：通常是JSON格式的完整响应
- **throws Exception**：同步部分的异常（如参数校验失败）

### 2. 典型使用场景
```java
// 1. 发起异步请求
CompletableFuture<String> future = openAiSession.completions(request);

// 2. 注册回调
future.thenAccept(response -> {
    // 3. 收到响应后处理
    ChatCompletionResponse res = JSON.parseObject(response, ChatCompletionResponse.class);
    updateUI(res);
});

// 4. 可以继续执行其他代码（不会被阻塞）
```

### 3. 完整示例
```java
public void askAI(String question) {
    ChatCompletionRequest request = new ChatCompletionRequest();
    request.setPrompt(Collections.singletonList(new Prompt(Role.USER, question)));
    
    openAiSession.completions(request)
        .thenApply(response -> JSON.parseObject(response, ChatCompletionResponse.class))
        .thenAccept(this::displayResponse)
        .exceptionally(e -> {
            showError("请求失败: " + e.getMessage());
            return null;
        });
}
```

## 五、与 RxJava 的关系

### 1. 相同点
- 都是异步编程工具
- 都支持链式调用
- 都能处理背压(backpressure)

### 2. 主要区别
| 特性     | CompletableFuture | RxJava     |
| -------- | ----------------- | ---------- |
| 来源     | Java 8 原生       | 第三方库   |
| 核心概念 | 单次异步计算      | 事件流     |
| 线程调度 | 简单              | 强大       |
| 学习曲线 | 平缓              | 陡峭       |
| 适用场景 | 简单异步任务      | 复杂事件流 |

### 3. 选择建议
- 如果你的项目已经在用RxJava，可以继续使用
- 如果是新项目且需求简单，用 `CompletableFuture` 就够了
- 需要复杂事件流处理时再考虑RxJava

## 六、常见问题解答

### Q1: 如何等待多个异步调用完成？
```java
CompletableFuture<String> future1 = openAiSession.completions(request1);
CompletableFuture<String> future2 = openAiSession.completions(request2);

CompletableFuture.allOf(future1, future2)
    .thenRun(() -> {
        String result1 = future1.join();
        String result2 = future2.join();
        // 处理合并结果...
    });
```

### Q2: 如何设置超时？
```java
future.completeOnTimeout("默认值", 30, TimeUnit.SECONDS)
      .thenAccept(result -> { ... });
```

### Q3: 如何取消任务？
```java
future.cancel(true); // true表示中断正在执行的任务
```

## 七、从零实现示例

假设我们要模拟一个简单的异步AI调用：

```java
public CompletableFuture<String> fakeAICall(String input) {
    CompletableFuture<String> future = new CompletableFuture<>();
    
    new Thread(() -> {
        try {
            Thread.sleep(1000); // 模拟处理耗时
            future.complete("AI回复: " + input.toUpperCase());
        } catch (Exception e) {
            future.completeExceptionally(e);
        }
    }).start();
    
    return future;
}

// 使用
fakeAICall("你好")
    .thenAccept(System.out::println)
    .exceptionally(e -> {
        System.out.println("出错: " + e.getMessage());
        return null;
    });
```

`CompletableFuture` 让异步编程变得简单直观，是处理类似ChatGLM这种网络请求的理想选择。通过链式调用，你可以优雅地组织异步逻辑，避免回调地狱。