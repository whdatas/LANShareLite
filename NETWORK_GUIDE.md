# 🌐 跨网段访问指南

本文档详细说明如何在不同网段、虚拟机、容器等复杂网络环境下使用 LAN Share。

## 📚 目录

1. [网络基础](#网络基础)
2. [多网卡场景](#多网卡场景)
3. [虚拟机网络](#虚拟机网络)
4. [Docker/容器](#docker 容器)
5. [跨网段路由](#跨网段路由)
6. [防火墙配置](#防火墙配置)
7. [故障排查](#故障排查)

---

## 网络基础

### IP 地址分类

| 类型 | 范围 | 说明 |
|------|------|------|
| A 类私有 | 10.0.0.0 - 10.255.255.255 | 大型网络 |
| B 类私有 | 172.16.0.0 - 172.31.255.255 | 中型网络 |
| C 类私有 | 192.168.0.0 - 192.168.255.255 | 家用/小型网络 |
| 回环 | 127.0.0.0 - 127.255.255.255 | 本地测试 |

### 子网掩码

- **255.255.255.0** (/24) — 最常见，254 个可用 IP
- **255.255.0.0** (/16) — 65,534 个可用 IP
- **255.0.0.0** (/8) — 16,777,214 个可用 IP

### 同网段 vs 跨网段

- **同网段**: 直接通信，无需路由器
- **跨网段**: 需要路由器/网关转发

---

## 多网卡场景

### 场景描述

你的电脑有多个网络接口：
- 以太网：192.168.1.100/24
- WiFi: 192.168.50.100/24
- 虚拟机网络：192.168.56.1/24

### 解决方案

```bash
# 1. 查看所有 IP
./lan-share-lite -all-ips

# 输出示例:
# 📍 网段 192.168.1.0/255.255.255.0:
#    • en0 (255.255.255.0) -> http://192.168.1.100:8080
# 📍 网段 192.168.50.0/255.255.255.0:
#    • en1 (255.255.255.0) -> http://192.168.50.100:8080
# 📍 网段 192.168.56.0/255.255.255.0:
#    • vboxnet0 (255.255.255.0) -> http://192.168.56.1:8080

# 2. 绑定到特定网卡
./lan-share-lite -bind 192.168.56.1

# 3. 监听所有接口（默认）
./lan-share-lite -bind 0.0.0.0
```

### 访问方式

- 192.168.1.x 网段的设备访问：`http://192.168.1.100:8080`
- 192.168.50.x 网段的设备访问：`http://192.168.50.100:8080`
- 192.168.56.x 网段的设备访问：`http://192.168.56.1:8080`

---

## 虚拟机网络

### VMware / VirtualBox 网络模式

| 模式 | IP 来源 | 外部访问 | 适用场景 |
|------|--------|----------|----------|
| **桥接** | 物理网络 DHCP | ✅ 可直接访问 | 推荐用于文件共享 |
| **NAT** | 虚拟网络 DHCP | ❌ 需端口转发 | 仅需上网 |
| **Host-Only** | 虚拟网络 DHCP | ⚠️ 仅主机可访问 | 隔离测试 |

### 桥接模式配置（推荐）

**VMware:**
1. 虚拟机设置 → 网络适配器
2. 选择"桥接模式"
3. 启动虚拟机，获取与主机同网段的 IP

**VirtualBox:**
1. 设置 → 网络
2. 连接方式：桥接网卡
3. 选择正确的物理网卡

### NAT 模式端口转发

如果必须使用 NAT 模式：

**VirtualBox:**
```bash
# 添加端口转发规则
VBoxManage modifyvm "VM Name" --natpf1 "share,tcp,,8080,,8080"
```

**VMware:**
编辑 `vmnetcfg.exe` 添加端口转发

### 在虚拟机内运行

```bash
# 虚拟机内查看 IP
ip addr show  # Linux
ipconfig      # Windows

# 启动服务
./lan-share-lite -bind 0.0.0.0

# 宿主机访问
http://<虚拟机 IP>:8080
```

---

## Docker/容器

### 方案 1: Host 网络（最简单）

```bash
docker run --network host my-image
```

容器内的服务直接绑定到主机网络，无需端口映射。

### 方案 2: 端口映射

```bash
docker run -p 8080:8080 my-image
```

宿主机访问：`http://localhost:8080`  
其他设备访问：`http://<宿主机 IP>:8080`

### 方案 3: 自定义网络

```bash
# 创建网络
docker network create mynet

# 运行容器
docker run --network mynet --ip 172.20.0.10 my-image
```

### ⚠️ Docker 中的 mDNS (Bonjour) 注意事项

为了让局域网能通过 `http://lan-share-lite.local:8080` 访问容器内的服务，需要注意：

1. **必须使用 Host 网络模式** (`--network host`): 
   - mDNS 需要通过 UDP 5353 端口在局域网广播。普通的端口映射模式下，广播流量会被 Docker 的网络隔离层阻挡。
   - **注意**: 此模式目前仅在 **Linux** 宿主机上完美支持。

2. **Docker Desktop (Mac/Windows) 的局限性**: 
   - 在 Mac/Windows 上，`--network host` 实际上是绑定到虚拟机的 IP 而非宿主机的网卡。
   - 因此，**在 Docker Desktop 环境下通常无法使用 mDNS 网址访问**，建议在此类环境下直接以二进制方式运行程序，或手动使用宿主机 IP 访问。

---

## 跨网段路由

### 场景

- 服务器：192.168.1.100/24
- 客户端：192.168.2.100/24
- 路由器：192.168.1.1 和 192.168.2.1

### 确保路由可达

```bash
# 在客户端检查路由
netstat -rn
# 或
ip route

# 应该有默认网关指向路由器
```

### 测试连通性

```bash
# ping 测试
ping 192.168.1.100

# 端口测试
telnet 192.168.1.100 8080
# 或
nc -zv 192.168.1.100 8080
```

---

## 防火墙配置

### macOS

```bash
# 查看防火墙状态
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# 允许 Go 或特定应用
# 系统偏好设置 → 安全性与隐私 → 防火墙 → 防火墙选项
# 添加 lan-share-lite 或 Go

# 或者临时关闭防火墙（仅测试用）
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off
```

### Linux (UFW)

```bash
# 查看状态
sudo ufw status

# 允许端口
sudo ufw allow 8080/tcp

# 重新加载
sudo ufw reload
```

### Linux (Firewalld)

```bash
# 查看状态
sudo firewall-cmd --state

# 允许端口
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### Windows

1. 控制面板 → Windows Defender 防火墙
2. 允许应用通过防火墙
3. 添加 lan-share-lite.exe 或 go.exe
4. 勾选私有和公共网络

---

## 故障排查

### 问题 1: ping 不通

```bash
# 检查 IP 是否正确
ipconfig / all  # Windows
ifconfig        # macOS/Linux

# 检查防火墙
# 临时关闭防火墙测试

# 检查物理连接
# 网线是否插好，WiFi 是否连接
```

### 问题 2: ping 通但无法访问网页

```bash
# 检查服务是否运行
lsof -i :8080  # macOS/Linux
netstat -ano | findstr :8080  # Windows

# 检查端口监听地址
# 应该是 0.0.0.0:8080 或具体 IP，而不是 127.0.0.1:8080

# 检查防火墙
# 确保 8080 端口被允许
```

### 问题 3: 同网段可以，跨网段不行

```bash
# 检查路由器配置
# 确保两个网段之间有路由

# 检查中间防火墙
# 企业网络可能有 ACL 限制

# 使用 traceroute 查看路径
traceroute 192.168.1.100  # macOS/Linux
tracert 192.168.1.100     # Windows
```

## 快速参考

### 常用命令

```bash
# 查看所有 IP
./lan-share-lite -all-ips

# 绑定特定 IP
./lan-share-lite -bind 192.168.1.100

# 监听所有接口
./lan-share-lite -bind 0.0.0.0

# 自定义端口
./lan-share-lite -port 9000

# 运行诊断
./diagnose.sh  # macOS/Linux
diagnose.bat   # Windows
```

### 访问地址格式

| 场景 | 访问地址 |
|------|----------|
| 同网段 | `http://192.168.1.100:8080` |
| 跨网段 | `http://<服务器 IP>:8080` |
| 虚拟机 | `http://<虚拟机 IP>:8080` |
| Docker | `http://<宿主机 IP>:8080` |

---

## 安全建议

1. **仅在可信网络使用** — 公共 WiFi 风险高
2. **使用强密码** — 如果添加认证功能
3. **定期更新** — 保持最新版本
4. **监控日志** — 注意异常访问
5. **用完即停** — 临时共享后及时关闭

---

如有问题，请运行诊断脚本或查看项目 README。
