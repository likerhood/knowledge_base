# http请求鉴权和postman使用

[TOC]

## 一、为什么需要鉴权？

- 大模型 API 通常需要鉴权，确保只有有权限的用户才能访问接口，防止滥用和保护数据安全。
- 鉴权方式主要有两种：API Key 认证 和 Token（JWT）认证。

---

## 二、API Key 认证

### 1. 获取 API Key

- 登录智谱AI开放平台，进入 API Keys 页面，生成你的 API Key。
- API Key 形如：`xxxxxx.yyyyyy`（前半部分是 id，后半部分是 secret）。

### 2. 使用 API Key 认证

- 在 HTTP 请求头（Header）中添加如下内容：
  ```
  Authorization: Bearer <你的apikey>
  ```
  例如：
  ```
  Authorization: Bearer 123456.abcdefg
  ```

### 3. curl 命令示例

```bash
curl --location 'https://open.bigmodel.cn/api/paas/v4/chat/completions' ^
--header 'Authorization: Bearer 1fa47f123902470890d81fdfcd249e4d.oaxJHknE0x0zFy66' ^
--header 'Content-Type: application/json' ^
--data '{
    "model": "glm-4",
    "messages": [
        {
            "role": "user",
            "content": "你好"
        }
    ]
}'
```

---

## 三、Token（JWT）认证

### 1. 为什么用 Token？

- Token 认证更安全，可以自定义有效期，适合对安全性要求更高的场景。

### 2. Token 生成方式

- 需要用代码（如 Python、Java 等）生成 JWT Token。
- Token 里包含你的 API Key id、过期时间、时间戳等信息，并用 API Key 的 secret 签名。

#### Python 生成 Token 示例

```python
import time
import jwt

def generate_token(apikey: str, exp_seconds: int):
    id, secret = apikey.split(".")
    payload = {
        "api_key": id,
        "exp": int(round(time.time() * 1000)) + exp_seconds * 1000,
        "timestamp": int(round(time.time() * 1000)),
    }
    return jwt.encode(
        payload,
        secret,
        algorithm="HS256",
        headers={"alg": "HS256", "sign_type": "SIGN"},
    )
```

### 3. 使用 Token 认证

- 在 HTTP 请求头中添加如下内容：
  ```
  Authorization: Bearer <你的token>
  ```

- 其余请求参数与 API Key 认证一致。

---

## 四、Postman 用法

### 1. Postman 是什么？

- Postman 是一个常用的 API 调试和测试工具，可以方便地构造、发送 HTTP 请求，并查看响应结果。

### 2. 如何用 Postman 调用大模型接口？

1. **选择 POST 方法，填入接口 URL**

2. **Headers 页签**
   - 添加一行：
     - Key: `Authorization`
     - Value: `Bearer 你的apikey` 或 `Bearer 你的token`
   - 如需指定数据格式，再加一行：
     - Key: `Content-Type`
     - Value: `application/json`
   
3. **Body 页签**
   - 选择 `raw`，格式选 `JSON`
   - 填写你的请求 JSON，例如：
     ```json
     {
       "model": "glm-4",
       "messages": [
         {"role": "user", "content": "你好"}
       ]
     }
     ```
   
4. **点击 Send 发送请求，查看响应**

![1745036382177](img/1745036382177.png)

![1745036423212](img/1745036423212.png)





## 五、总结

- **API Key 认证**：简单直接，适合大多数场景。
- **Token 认证**：更安全，适合有特殊安全需求的场景。
- **curl 和 Postman** 都可以用来测试接口，关键是正确设置 Authorization 头部
