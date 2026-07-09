# localhost 能打开，其他电脑打不开？完整排查 8080 端口

这是局域网共享最典型的问题：服务端打开 `http://localhost:8080` 完全正常，另一台电脑访问 `http://192.168.x.x:8080` 却超时。

这通常说明程序已经启动，问题位于监听地址、防火墙、网络类型或路由，而不是网页本身。

## 先理解 localhost 的含义

`localhost` 和 `127.0.0.1` 都代表“当前这台机器”。本机访问成功只能证明：

- 程序正在运行；
- HTTP 服务能响应本机请求；
- 端口在回环网络中可用。

它不能证明外部设备能进入这台电脑。

## 第一步：检查监听地址

Windows：

```powershell
netstat -ano | findstr ":8080"
```

Linux/macOS：

```bash
ss -lntp | grep 8080
# 或
lsof -i :8080
```

期望看到：

```text
0.0.0.0:8080
[::]:8080
```

如果只看到 `127.0.0.1:8080`，检查 `app.yaml`：

```yaml
bind: ""
port: 8080
```

空 bind 表示监听所有网卡。

## 第二步：确认使用了正确 IP

Windows 运行 `ipconfig`，Linux/macOS 运行 `ip addr` 或 `ifconfig`。选择和客户端同网段的地址。

例如客户端是 `192.168.1.50`，服务端通常也应使用 `192.168.1.x`，而不是 VPN、Docker、VMware 或 VirtualBox 的虚拟地址。

## 第三步：检查端口连通性

在另一台 Windows 电脑运行：

```powershell
Test-NetConnection 192.168.1.100 -Port 8080
```

若 `TcpTestSucceeded` 为 `False`，浏览器自然也无法访问。先解决网络层，不必反复清缓存或重装浏览器。

## 第四步：检查 Windows 防火墙与网络类型

```powershell
netsh advfirewall firewall show rule name="lan-share-lite HTTP"
Get-NetConnectionProfile
```

安装程序默认创建 Private/Domain 入站规则。若当前 Wi-Fi 被标记为 Public，规则可能不生效。确认这是可信家庭或办公网络后，可改为 Private：

```powershell
Set-NetConnectionProfile -InterfaceAlias "Wi-Fi" -NetworkCategory Private
```

不要为了省事永久关闭整个 Windows 防火墙。

## 第五步：检查路由器与网络隔离

如果两台设备都连同一个 Wi-Fi 仍不通，检查：

- 路由器是否开启 AP/客户端隔离；
- 一台连接访客网络、另一台连接主网络；
- 公司网络 VLAN 是否禁止横向访问；
- VPN 是否接管了默认路由；
- 虚拟机使用 NAT 而不是桥接；
- Windows 防病毒软件是否自带网络防护。

## 使用内置连通性自检

服务端打开：

```text
http://localhost:8080/diagnostics
```

自检页会显示监听地址、局域网网卡、可测试 URL、访问来源和 Windows 防火墙规则提示。

真正有效的外部验证是：在另一台设备打开：

```text
http://服务端IP:8080/diagnostics
```

如果该页面能从另一台设备打开，就说明至少这条客户端到服务端的 HTTP 入站链路正常。

## 快速判断表

| 现象 | 更可能的原因 |
|---|---|
| localhost 也打不开 | 服务未启动、端口冲突、配置错误 |
| localhost 正常，IP 本机也打不开 | bind 只监听回环或选错 IP |
| 本机 IP 正常，其他设备不通 | 防火墙、Public 网络、路由隔离 |
| 有些设备能访问，有些不能 | 不同网段、访客网络、设备策略 |
| 小文件正常，大文件中断 | Wi-Fi 稳定性、超时、磁盘空间 |

Lite版本 可以帮助判断本机状态，但无法绕过企业 VLAN 和路由策略。需要多节点、集中运维和告警时，应由 Pro/Enterprise 配合正式网络规划。

下一篇：[拖拽上传、目录上传与冲突处理](08-upload-and-conflicts.md)

[<< 返回系列目录](../README.md)
