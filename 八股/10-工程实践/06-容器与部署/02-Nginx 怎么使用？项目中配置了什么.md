---
aliases:
- "Nginx 怎么使用？项目中配置了什么？"
- "工具与工程 3.2 Nginx 怎么使用？项目中配置了什么？"
---

# 02 Nginx 怎么使用？项目中配置了什么？

## 01 核心回答


**概念原理：**Nginx 是高性能反向代理/Web 服务器，事件驱动 + 多 worker 进程，单机可扛十万级并发连接。项目中的角色：反向代理（转发到后端服务）、静态资源服务、负载均衡、SSL 终结、限流。核心配置如下：

```nginx
# ① 反向代理
location /api/ {
    proxy_pass http://backend;
}

# ② 负载均衡（轮询 / 加权 / IP hash）
upstream backend {
    server 10.0.0.1:8080 weight=3;
    server 10.0.0.2:8080;
    # ip_hash;  按客户端 IP 哈希，会话保持
}

# ③ 静态资源 + gzip 压缩
location /static/ {
    root /data/web;
    expires 7d;          # 强缓存一周
    gzip on;
}

# ④ SSL 终结
server {
    listen 443 ssl;
    ssl_certificate     /etc/nginx/cert.pem;
    ssl_certificate_key /etc/nginx/cert.key;
}

# ⑤ 限流（每 IP 10 r/s，burst 20 个突发，nodelay 使允许的突发不排队）
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
location /api/ {
    limit_req zone=api burst=20 nodelay;
}

# ⑥ 超时与健康检查
proxy_connect_timeout 5s;
proxy_read_timeout 60s;
```

**关键细节：**反向代理要透传真实 IP（`proxy_set_header X-Real-IP $remote_addr;`）；WebSocket 需要 `Upgrade` 头；静态资源命中缓存减少后端压力；gzip 压缩传输体积。

**风险与取舍：**Nginx 是单点（前面加 LVS/云 LB 或 Nginx 集群）；配置错误会导致服务不可用（先 `nginx -t` 验证再 reload）；连接数和 worker 数按机器调整（`worker_processes`、`worker_connections`）。

**项目落地：**前端静态资源 + 后端 API 同域部署（避免跨域）；按路径分流（`/api` 到后端、其他走静态）；灰度：upstream 里新旧版本按权重切换；限流保护后端。

**面试追问：**Nginx 为什么快（事件驱动 + 非阻塞 + 零拷贝）；反向代理和正向代理的区别（服务端 vs 客户端代理）；reload 为什么不断连接（平滑重载）；Nginx 和 LVS 的区别（七层 vs 四层）。

---

## 02 理解补充与边界校订

旧配置里 nodelay 的含义需订正：允许 burst 范围内请求不等待，超出允许突发才被拒绝，并非所有超额请求立即拒绝。示例的 upstream 名称也不一致，部署前要保证 proxy_pass 指向真实定义。proxy_read_timeout 限制连续两次读取间隔，不能当作整个请求总期限；反向代理的重试也要考虑写请求重复执行。

## 03 依据与延伸阅读

- [Nginx limit_req](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)
- [Nginx proxy 超时语义](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)

## 04 相关问题

- [[八股/10-工程实践/06-容器与部署/04-项目是怎么部署的|项目是怎么部署的]]：入口代理与部署流程

## 05 相关问题与延伸

- [[八股/11-前端/04-HTTP安全与集成/10-Nginx 在前端项目中承担什么作用？如何配置？（关联 工具与工程 3.2）|Nginx 在前端项目中承担什么作用？如何配置？（关联 工具与工程 3.2）]]：前端部署入口与代理配置

## 06 所属专题

- [[八股/10-工程实践/06-容器与部署/00-容器与部署导航|容器与部署导航]]
