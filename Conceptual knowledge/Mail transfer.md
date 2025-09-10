# 邮件传输原理

## 一、邮件相关的 DNS 记录

在邮件传输中，DNS 不只是做 A/AAAA 解析，还承担了邮件路由和安全验证功能。主要有以下几类记录：

### MX (Mail Exchanger)

指定接收该域名邮件的服务器地址。

example.com.   MX 10 mail.protonmail.ch.
### SPF（存放在 TXT 中）

定义哪些服务器有权限代表该域发送邮件。

收件方通过检查发件 IP 是否匹配 SPF 规则来防止伪造。

```
v=spf1 include:_spf.protonmail.ch include:amazonses.com ~all
```
### DKIM（TXT 或 CNAME 形式）

域名签名机制，公钥存放在 DNS 中。

发件服务器对邮件进行签名，收件服务器通过 DNS 公钥验证完整性与来源合法性。

```
"v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAiweykoi+o48IOGuP7GR3X0MOExCUDY/BCRHoWBnh3rChl7WhdyCxW3jgq1daEjPPqoi7sJvdg5hEQVsgVRQP4DcnQDVjGMbASQtrY4WmB1VebF+RPJB2ECPsEDTpeiI5ZyUAwJaVX7r6bznU67g7LvFq35yIo4sdlmtZGV+i0H4cpYH9+3JJ78k" "m4KXwaf9xUJCWF6nxeD+qG6Fyruw1Qlbds2r85U9dkNDVAS3gioCvELryh1TxKGiVTkg4wqHTyHfWsp7KD3WQHYJn0RyfJJu6YEmL77zonn7p2SRMvTMP3ZEXibnC9gz3nnhR6wcYL8Q7zXypKTMD58bTixDSJwIDAQAB"
```
### DMARC（TXT 记录）

基于 SPF 和 DKIM 的策略控制。

- 定义收件方遇到验证失败的处理方式（none/quarantine/reject）
- 也可配置报告地址用于监控冒用情况

```
"v=DMARC1; p=none; rua=mailto:2ab0fe6788124507adac7d96a9a76f7e@dmarc-reports.cloudflare.net"
```
## 二、TLS 在邮件传输中的作用与过程

邮件协议早期均为明文，TLS 的引入用于保证机密性与完整性。

### 客户端 ↔ 邮件服务器

发送（SMTP Submission，587/465）和接收（IMAPS 993/POP3S 995）均应使用 TLS。

两种模式：

- **隐式 TLS**：连接建立后立即进入 TLS（如 SMTPS 465，IMAPS 993）
- **STARTTLS**：先明文建立连接，再通过 STARTTLS 命令升级到加密通道（如 SMTP 587，SMTP 25 服务器间传输）

### 握手过程（简化版 TLS 1.3）

1. ClientHello/ServerHello：协商加密算法与密钥交换方式
2. 服务器发送证书，客户端验证证书链与主机名
3. 双方完成密钥交换，进入加密通道
4. 后续 SMTP/IMAP 命令、用户名密码、邮件正文均在 TLS 内传输

### 服务器 ↔ 服务器（MTA-MTA）

- 默认 25 端口，SMTP + STARTTLS（若对方支持）
- 若配置了 MTA-STS 或 DANE，可强制要求加密及证书校验，否则是机会性 TLS

## 三、SMTP 与 IMAP 的职责与工作原理

### SMTP（Simple Mail Transfer Protocol）

用于 **发送邮件**：

- 客户端（MUA）提交邮件到发件服务器（MSA/MTA）
- 发件服务器通过 DNS MX 查询找到目标域的接收服务器，并完成邮件转发
- 基于信封命令：`MAIL FROM`、`RCPT TO`、`DATA`
- 本质上负责 **邮件投递链路**

### IMAP（Internet Message Access Protocol）

用于 **接收和管理邮件**：

- 客户端连接收件服务器，从服务器上读取、同步邮件
- 支持多端同步（标记已读/未读、文件夹管理）
- 相比 POP3，不会默认删除服务器端邮件，支持在线操作和多设备访问

## 四、整体流程（概览）

### 发件人客户端 → 发件服务器

使用 SMTP + TLS（587/465），客户端认证后提交邮件。

### 发件服务器 → 收件服务器

发件服务器查询 MX 记录，通过 SMTP（25，通常 STARTTLS）将邮件投递到目标域的接收 MTA。

收件服务器进行 SPF/DKIM/DMARC 检查，若通过则接受并存储邮件。

### 收件人客户端 → 收件服务器

使用 IMAP + TLS（993）从服务器读取邮件，保持状态同步。

---

## 总结

✅ 这样你就能看到：

- **DNS 记录** 决定路由与验证
- **TLS** 负责传输加密与身份验证
- **SMTP** 管理邮件的"发送与传递"
- **IMAP** 管理邮件的"存储与读取"
