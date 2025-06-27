# nginx笔记

[TOC]



## 一、Nginx 是什么？
### 1. 基础认知
• **Web服务器**：可直接托管网站文件（HTML/CSS/JS/图片）
• **反向代理服务器**：像"中间商"转发请求到后端服务（Node.js/Java/Python等）
• **负载均衡器**：把流量分配给多个服务器
• **高性能**：C语言编写，事件驱动架构，单机可处理10万+并发连接

### 2. 常见使用场景
```
┌───────────────┐       ┌───────────────┐
│   客户端       │       │   Nginx       │
│ (浏览器/APP)   │─请求─▶│               │
└───────────────┘       ├───────────────┤
                        │ ▸ 静态文件处理  │
                        │ ▸ 反向代理     │
                        │ ▸ 负载均衡     │
                        │ ▸ SSL加密      │
                        └───────────────┘
                               ▼
                         ┌───────────────┐
                         │ 真实服务器集群  │
                         │ (Node/Java等) │
                         └───────────────┘
```





## 二、反向代理 vs 正向代理

### 1. 反向代理（Reverse Proxy）
• **隐藏真实服务器**：客户端不知道真正提供服务的是谁
• **企业级应用**：淘宝/京东等大型网站都在使用
• **配置示例**：
  ```nginx
  location / {
      proxy_pass http://127.0.0.1:3000;  # 转发到本地的Node.js服务
  }
  ```

### 2. 正向代理（Forward Proxy）
• **隐藏客户端**：服务器不知道真正的请求方是谁
• **常见用途**：科学上网/VPN访问外网
• **典型代表**：Shadowsocks/V2Ray





## 三、Nginx 基础配置

### 1. 核心配置文件结构
```nginx
# 全局配置（影响所有配置）
user nginx;                     # 运行用户
worker_processes auto;          # 工作进程数（CPU核数）

events {                        # 连接配置
    worker_connections 1024;    # 单个进程最大连接数
}

http {                          # HTTP协议配置
    server {                    # 虚拟主机配置
        listen 80;              # 监听端口
        server_name example.com; # 域名,必须要有http头
        
        location / {            # 路由规则
            root /var/www/html; # 网站根目录
            index index.html;   # 默认首页
        }
    }
}
```

### 2. 核心指令速查表
| 指令          | 作用说明         | 示例值                      |
| ------------- | ---------------- | --------------------------- |
| `listen`      | 监听端口         | `80`, `443 ssl`             |
| `server_name` | 匹配的域名       | `example.com`, `*.test.com` |
| `root`        | 网站根目录       | `/var/www/html`             |
| `index`       | 默认首页文件     | `index.html`                |
| `proxy_pass`  | 反向代理转发地址 | `http://localhost:3000`     |
| `error_page`  | 自定义错误页面   | `error_page 404 /404.html;` |





## 四、Location 配置详解

### 1. 匹配规则优先级表（重要！）
| 匹配类型         | 符号   | 优先级 | 示例                      |
| ---------------- | ------ | ------ | ------------------------- |
| 精确匹配         | `=`    | 1      | `location = /login`       |
| 前缀匹配         | `^~`   | 2      | `location ^~ /static/`    |
| 正则匹配         | `~`    | 3      | `location ~ \.php$`       |
| 不区分大小写正则 | `~*`   | 4      | `location ~* \.(jpg|png)` |
| 普通前缀匹配     | 无符号 | 5      | `location /`              |

### 2. 常见配置模板
#### 场景1：静态文件处理
```nginx
location ^~ /static/ {
    root /var/www;          # 实际路径：/var/www/static/
    expires 7d;             # 客户端缓存7天
    gzip on;                # 启用压缩
}
```

#### 场景2：API接口代理
```nginx
location /api/ {
    proxy_pass http://backend:8000/;  # 注意结尾斜杠！
    proxy_set_header Host $host;      # 传递原始域名
}
```

#### 场景3：图片防盗链
```nginx
location ~* \.(jpg|png)$ {
    valid_referers none blocked *.example.com;
    if ($invalid_referer) {
        return 403;
    }
}
```







### 五、windows的docker环境

1. 下载docker desktop
2. 在docker egine设置的镜像源，注意不要协商data-root配置