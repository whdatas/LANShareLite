# macOS、Linux 与 Docker 部署 LAN Share Lite

LAN Share Lite 的访问端只需要浏览器，服务端则可以按环境选择原生安装或容器部署。个人 Mac、办公室 Linux 服务器、NAS 和小主机不必使用同一种方式。

## macOS：适合桌面电脑常驻共享

下载 `.pkg` 后双击安装。若 macOS 提示无法验证开发者，可以在“系统设置 -> 隐私与安全性”中选择仍要打开，或在 Finder 中右键选择“打开”。

安装后服务通过 LaunchDaemon 运行，常见路径为：

```text
配置：/etc/lan-share-lite/app.yaml
数据：/Library/Application Support/lan-share-lite/user_data
```

检查服务：

```bash
sudo launchctl list | grep lanshare
```

卸载时使用安装目录内的官方卸载脚本，默认保留用户数据。

## Debian/Ubuntu：使用 DEB 包

```bash
sudo dpkg -i lan-share-lite_*_amd64.deb
sudo systemctl status lan-share-lite
```

主要目录：

```text
配置：/etc/lan-share-lite/app.yaml
数据：/var/lib/lan-share-lite/user_data
日志：/var/log/lan-share-lite
```

修改配置后：

```bash
sudo systemctl restart lan-share-lite
```

## RHEL/CentOS/Fedora：使用 RPM 包

```bash
sudo rpm -ivh lan-share-lite-*.rpm
sudo systemctl status lan-share-lite
```

如果运行环境没有 systemd，例如某些容器或精简系统，安装脚本会跳过服务注册，此时可手动运行程序。

## Docker：适合 NAS 和服务器

Docker 的关键不是启动命令，而是把数据目录正确映射到宿主机：

```yaml
services:
  lan-share-lite:
    image: harbor.whdatas.com/lans/lan-share-lite:latest
    container_name: lan-share-lite
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./share:/app/user_data
      - ./configs:/app/configs:ro # 可选：把 configs 也映射出来，方便修改配置后重启容器, 如果不是很熟悉docker同学，建议先注释掉这一行跑起来
```

启动与查看日志：

```bash
docker compose up -d
docker compose logs -f lan-share-lite
```

如果忘记映射 `/app/user_data`，容器删除后文件也会随容器层丢失。生产使用前务必在宿主机验证文件实际落盘。

## 三种方式怎么选

| 环境 | 推荐方式 | 原因 |
|---|---|---|
| 个人 Mac | PKG | 安装直观、自动常驻 |
| Linux 服务器 | DEB/RPM | 便于 systemd 管理和日志排查 |
| NAS/已有容器平台 | Docker Compose | 配置和迁移更统一 |
| 临时测试 | 直接运行二进制 | 最少步骤 |

## 部署后统一验证

无论哪种方式，都执行三项检查：

1. 本机打开 `http://localhost:8080`；
2. 确认页面显示正确的数据目录；
3. 用另一台设备打开 `http://服务端IP:8080/diagnostics`。

## 如何卸载
```bash
# for macOS
sudo /Library/Application\ Support/lan-share-lite/uninstall.sh

# for Debian/Ubuntu
sudo dpkg -r lan-share-lite

# for RHEL/CentOS/Fedora
sudo rpm -e lan-share-lite
```

下一篇：[第一次使用 LAN Share Lite](06-first-use.md)

[<< 返回系列目录](../README.md)
