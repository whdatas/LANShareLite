# 📁 LAN Share - 跨平台局域网文件共享

> **🌍 官方网站**: [https://lan-share.whdatas.com](https://lan-share.whdatas.com/lite-version)

用 Go 实现的轻量级局域网文件共享工具，支持 macOS、Windows、Linux。

[![Tests](https://github.com/jackman0925/lan-share-lite/actions/workflows/ci.yml/badge.svg)](https://github.com/jackman0925/lan-share-lite/actions/workflows/ci.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/jackman0925/lan-share-lite)](https://goreportcard.com/report/github.com/jackman0925/lan-share-lite)
[![Go Version](https://img.shields.io/badge/go-1.23-00ADD8?logo=go)](https://golang.org/)
[![Platform](https://img.shields.io/badge/platform-macOS%7Cwindows%7Clinux-lightgrey)](README.md)

## ✨ 特性

- **🌐 跨平台** — 一次编译，到处运行
- **🔍 自动发现 (mDNS/Bonjour)** — 局域网内通过 `http://lan-share-lite.local:8080` 网址直接访问，无需记 IP
- **📱 全平台访问** — 浏览器、Finder、文件资源管理器都能用
- **📤 上传下载** — 支持拖拽上传、多文件上传
- **🎨 美观界面** — 响应式设计，手机电脑都能用
- **🔒 安全** — 目录限制、可选删除保护
- **⏱️ 超时控制** — 可配置读写超时，避免大文件传输中断

## 🚀 快速开始

### 1.1. Docker 部署 (推荐)

如果您更倾向于使用 Docker，我们提供了深度优化的多架构镜像：

```bash
docker run -d -p 8080:8080 -v $(pwd):/app/user_data --name lan-share-lite harbor.whdatas.com/lans/lan-share-lite:latest
```

更多优化和多平台构建详情，请参阅：[Docker 部署与优化指南](docs/DOCKER.md)。

### 1.2. 原生态安装 (最快)

根据系统来下载安装包，目前支持 Windows macOS Linux(rpm, deb)

### 2. 访问

运行后会显示访问地址：

```
访问方式:
   • http://lan-share-lite.local:8080 (推荐，支持 mDNS 设备)
   • http://192.168.0.101:8080
```

**各平台访问方式：**

| 平台 | 访问方式 |
|------|----------|
| **macOS** | Finder → ⌘+K → 输入 `http://lan-share-lite.local:8080` |
| **Windows** | 文件资源管理器 → 地址栏输入 IP 或 http://lan-share-lite.local:8080 |
| **Linux** | 浏览器或文件管理器 (Nautilus/Dolphin) |
| **手机/平板** | 浏览器直接访问 |
| **任意设备** | 浏览器访问任意显示的 IP 或 mDNS 地址 |


## 日志配置

日志配置来自 `configs/logger.yaml`，可调整级别和输出位置。

## 配置文件

支持自动查找配置文件（`/etc/lan-share-lite/app.yaml` 或 `configs/app.yaml`），也可通过 `-config` 指定路径。  
生效顺序：默认值 → 配置文件 → 命令行参数。

可配置项示例（见 `configs/app.yaml`）：

- `dir` 共享目录（空值→自动；绝对路径→直接使用；相对路径→拼接到默认目录）
- `port` 端口
- `name` 服务名称
- `bind` 绑定地址
- `enable_upload` 是否允许上传
- `enable_delete` 是否允许删除
- `show_all_ips` 是否显示所有 IP
- `max_upload_size` 最大上传大小（如 `10GB`）
- `write_timeout` 写超时（秒）
- `read_timeout` 读超时（秒）
- `temporary` 是否临时共享（过期后自动清理）
- `temp_ttl` 临时共享有效期（如 `24h`）
- `preserve` 永久保留列表（如 `readme.txt`, `.gitkeep`, `docs/**`）
- `header_theme` 头部主题（`teal` | `neon` | `sunset`）

说明：即使非容器安装，也不会受影响；配置文件是可选项，不提供也能正常使用默认值。


## 更多文档

- **[跨网段访问指南](NETWORK_GUIDE.md)** — 详细的网络配置和故障排查
- **[大文件传输指南](LARGE_FILE_GUIDE.md)** — GB 级文件传输优化和最佳实践

## 安全提示

1. **仅在可信网络使用** — 不要在公共 WiFi 上运行
2. **默认禁用删除** — 防止误操作
3. **目录限制** — 无法访问共享目录外的文件
4. **无认证** — 如需认证，可添加基础 HTTP Auth
5. **防火墙** — 确保端口未被阻挡

## 常见问题

### macOS 安装提示“无法验证恶意软件”？

由于安装包未经过 Apple 开发者签名，macOS 会拦截运行。请尝试以下任一方法：

1. **右键打开：** 在 Finder 中对 `.pkg` 文件点击右键（或 Control+点击），选择“打开”。此时弹出的对话框会多出一个“打开”按钮。
2. **系统设置授权：** 前往“系统设置” -> “隐私与安全性” -> “安全性”，点击“仍要打开”。

## 故障排查

运行诊断脚本自动检查网络配置：

```bash
# macOS/Linux
./diagnose.sh

# Windows
diagnose.bat
```

### 手动排查

如果不同网段的设备无法访问：

### 1. 检查连通性
```bash
# 在客户端 ping 服务器 IP
ping 192.168.1.100

# 测试端口是否开放
telnet 192.168.1.100 8080
# 或
nc -zv 192.168.1.100 8080
```

### 2. 检查防火墙
```bash
# macOS
sudo lsof -i :8080

# Linux
sudo ufw status
sudo ufw allow 8080/tcp

# Windows
# 防火墙 → 允许应用通过防火墙 → 添加 lan-share-lite
```

### 3. 检查路由
```bash
# 查看路由表
netstat -rn
# 或
ip route

# 确保客户端和服务器之间有路由可达
```

### 4. 虚拟机/容器网络
- **VMware/VirtualBox**: 使用桥接模式而非 NAT
- **Docker**: 使用 `--network host` 或端口映射
- **WSL2**: 使用 WSL2 的 IP 而非 localhost

## 卸载

如果您需要卸载程序，请根据您的平台参考以下说明：

### macOS
```bash
# 推荐：直接运行安装好的卸载脚本
sudo "/Library/Application Support/lan-share-lite/uninstall.sh"

```

### Linux (DEB)
```bash
sudo dpkg --purge lan-share-lite
```

### Linux (RPM)
```bash
sudo rpm -e lan-share-lite
```

### Windows
通过 **控制面板 -> 卸载程序** 找到 `lan-share-lite` 进行卸载，或运行安装目录下的 `uninstall.exe`。

## 界面预览

- 文件浏览（带图标和大小）
- 面包屑导航
- 拖拽上传
- 文件夹上传
- 网页新建文件夹
- 响应式设计
