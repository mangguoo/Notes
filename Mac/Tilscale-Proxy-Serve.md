## 1) 为什么 127.0.0.1 的服务外部访问不到？和 0.0.0.0 有什么区别

127.0.0.1 是 loopback（回环）地址。服务如果 bind 到 `127.0.0.1:18927`，它只在本机回环网卡上“开门”，外部机器（哪怕在同一局域网或 Tailnet）发到你机器真实网卡/ Tailscale 网卡的流量，不会被内核转到 loopback 上，所以外部必然连不上。([Stack Overflow][1])

0.0.0.0（INADDR_ANY）在“监听地址”这个语境里表示“监听本机所有网卡的 IPv4 地址”。服务 bind 到 `0.0.0.0:18927` 时，来自 LAN、来自 Tailscale（100.x）、来自其它本机地址的连接都有机会打到这个端口（接不接受还取决于防火墙/ACL）。([Stack Overflow][1])

一句话：`127.0.0.1` 只给本机用；`0.0.0.0` 给所有网卡用（范围更大，风险也更大）。

## 2) 用 Tailscale Serve 解决“服务只监听 127.0.0.1”的问题

核心思路：不改你的服务监听地址，让它继续只在 `127.0.0.1:18927` 上跑；由 Tailscale Serve 在 Tailnet 内提供一个入口，然后把请求反向代理到 localhost。Tailscale 官方对 Serve 的定义就是“把 tailnet 里其他设备的流量路由到你设备上的本地服务”。([Tailscale][2])

你的场景（Mac mini，本地 Web 服务 18927）最直接的做法是：

1. 本机先确认服务确实在跑
   `curl -I http://127.0.0.1:18927`

2. 开启 Serve（Tailnet 内可访问）
   `sudo tailscale serve --bg http://127.0.0.1:18927` ([Tailscale][3])

之后，组内用户访问你这台设备的 Tailscale 名称（MagicDNS）或 Tailnet IP，就会被代理到 `127.0.0.1:18927`。Serve 的价值就在这里：让“只对本机开放”的服务安全地对 tailnet 开放。([Tailscale][2])

注意点：如果你们启用了 ACL，必须允许组内成员访问你这台设备对应的端口/服务，否则会出现“设备在线但端口不可达”。

## 3) tailscale serve 常用指令（查询/多映射/关闭）

以下以官方 `tailscale serve` 参考为准（1.52 之后 CLI 有过调整，建议以你当前版本 `tailscale serve --help` 为最终准绳）。([Tailscale][2])

查看当前 Serve 规则

* `tailscale serve status`
* `tailscale serve status --json`（方便脚本处理）([Tailscale][3])

创建/开启映射（示例）

* 最简单（把 tailnet 入口代理到本机 http 服务）：
  `sudo tailscale serve --bg http://127.0.0.1:18927` ([Tailscale][3])
  
* 映射多个：思路是“多个入口 -> 多个后端”，常见做法是用不同对外端口或不同路径分别指向不同本地端口

  例子（路径分流）：

  `tailscale serve --bg --set-path=/a 3000`

  `tailscale serve --bg --set-path=/b 4000`

  访问就是 https://<node>.ts.net/a、/b。--set-path 会把规则挂载到不同 URL path 上

关闭某一条映射

* 原则：对“同一个入口配置”执行 off（保留当初相同的入口参数），把该条规则移除:

  `tailscale serve --set-path=/a 3000 off`

清空所有 Serve 配置

* `sudo tailscale serve reset`（一键清掉本机全部 serve 规则）([Tailscale][3])

