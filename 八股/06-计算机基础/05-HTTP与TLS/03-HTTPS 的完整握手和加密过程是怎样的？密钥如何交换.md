---
aliases:
- "HTTPS 的完整握手和加密过程是怎样的？密钥如何交换？"
- "计算机网络 4.3 HTTPS 的完整握手和加密过程是怎样的？密钥如何交换？"
---

# 03 HTTPS 的完整握手和加密过程是怎样的？密钥如何交换？

## 01 核心回答


**概念原理：**HTTPS 用**混合加密**：非对称（RSA/ECDHE）只用于身份认证和密钥协商，协商出的对称密钥（AES-GCM）用于实际数据加密——兼顾安全与性能。

**TLS 1.2 的 ECDHE 完整握手（常见两轮往返，不含 TCP）：**客户端发送 ClientHello；服务端回复 ServerHello、Certificate、带签名的临时密钥交换参数 ServerKeyExchange、ServerHelloDone。客户端验证证书链、主机名及签名，再通过 ClientKeyExchange 发送自己的临时公钥份额。双方由各自私钥和对方公钥得到共享秘密；再派生 master secret 和记录层密钥，并非把 premaster 直接当作业务加密密钥。客户端发送 ChangeCipherSpec 和 Finished，服务端验证后发送自己的 ChangeCipherSpec 和 Finished；此处省略可选的客户端证书认证步骤。

**TLS 1.3 常见证书认证流程：**ClientHello 已携带 key_share；ServerHello 选定参数并带回服务端 key_share，双方据此派生握手密钥。服务端随后发送加密的 EncryptedExtensions、Certificate、CertificateVerify 和 Finished，客户端验证后发送 Finished，再使用派生的应用流量密钥通信。常规流程可在一轮往返后建立保护，HelloRetryRequest 等情况会增加交互；会话恢复也不自动等于 0-RTT，early data 必须被客户端尝试且由服务端允许和接受。

**密钥交换两种方式：**① **RSA 密钥交换**：客户端用服务器公钥加密 premaster 发给服务器——私钥泄露则历史流量全解密（无前向保密），已弃用；② **ECDHE**：双方用椭圆曲线 Diffie-Hellman 各自计算同一共享秘密，再派生流量密钥，**私钥不传输**——即使长期私钥泄露，历史会话仍安全（前向保密），TLS 1.3 支持（EC）DHE、PSK 及组合模式，不支持旧式 RSA 密钥传输。

**关键细节：**证书的作用是**把公钥绑定到域名**（防中间人替换公钥）；流量密钥由共享秘密、握手上下文和密钥派生过程产生；握手中的 Finished 认证握手转录并证明相应握手秘密的持有。

**风险与取舍：**2-RTT 握手延迟（用会话复用/0-RTT 优化）；0-RTT 有重放风险（只用于幂等请求）；证书私钥泄露是灾难（轮换 + 吊销）；中间设备（防火墙）可能干扰 TLS 1.3。

**项目落地：**网关终结 TLS（集中管理证书和改变连接/信任边界，并不天然减少客户端一次握手）；会话恢复凭据（如 session ticket）可减少后续握手成本，但仍需要恢复握手；证书自动化续期；安全扫描验证 TLS 版本和套件。

**面试追问：**前向保密是什么（ECDHE 保证历史流量不被长期私钥解密）；证书链验证过程（根 CA → 中间 CA → 叶子证书）；1-RTT 和 0-RTT 的区别；为什么不用非对称直接加密数据（性能差）。

---

## 02 理解补充与边界校订

TLS 1.3 支持（EC）DHE、PSK 以及二者组合，并非只允许 ECDHE；是否具有前向保密要看实际模式。协商得到共享秘密后还要通过密钥派生生成不同方向、不同阶段的流量密钥。0-RTT 只适用于恢复且服务端接受的场景，具有重放风险；业务幂等是必要考虑，但仍要结合认证、重放防护和处理上下文判断能否重放。

## 03 依据与延伸阅读

- [RFC 5246：TLS 1.2 消息序列与密钥计算](https://www.rfc-editor.org/rfc/rfc5246.html)
- [RFC 8446](https://www.rfc-editor.org/rfc/rfc8446.html)

## 04 相关问题

- [[八股/06-计算机基础/05-HTTP与TLS/01-HTTP 和 HTTPS 有什么区别|HTTP 和 HTTPS 有什么区别]]：安全属性与具体握手

## 05 所属专题

- [[八股/06-计算机基础/05-HTTP与TLS/00-HTTP与TLS导航|HTTP与TLS导航]]
