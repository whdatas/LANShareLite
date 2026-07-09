# Windows 10/11 安装 LAN Share Lite：从安装包到后台服务

在 Windows 上把一个程序“双击打开”不难，真正影响长期使用的是：它能否后台运行、开机启动、找到配置和数据目录，以及能否通过防火墙让其他电脑访问。

LAN Share Lite 的 Windows 安装包会把主程序安装到 Program Files，并可通过 NSSM 注册为 Windows 服务。

## 安装前准备

确认以下条件：

- Windows 10 或 Windows 11；
- 当前账号可以批准管理员权限；
- 服务端电脑与访问端在同一可信局域网；
- 默认端口 `8080` 没被其他程序占用。

从 [LAN Share 官网](https://lan-share.whdatas.com/lite-version) 下载 Windows 安装包，文件名类似：

```text
lan-share-lite-0.x.x-setup.exe
```

## 安装步骤

1. 右键安装包，选择“以管理员身份运行”；
2. 选择安装目录，默认是 `C:\Program Files\lan-share-lite`；
3. 确认勾选“安装为 Windows 服务”；
4. 完成安装；
5. 浏览器打开 `http://localhost:8080` 在非docker环境，可点击 “连通性自检” 按钮里面查看 "局域网测试地址"就是局域网内均可访问的地址，比如 http://192.168.0.101:8080。

安装后的主要路径：

```text
程序：C:\Program Files\lan-share-lite
配置：C:\ProgramData\lan-share-lite\configs
文件：C:\ProgramData\lan-share-lite\user_data
日志：C:\ProgramData\lan-share-lite\logs
```

## 检查后台服务

在管理员 PowerShell 中执行：

```powershell
Get-Service LANShareLite
sc.exe query LANShareLite
```

正常情况下状态应为 `Running`。需要重启时：

```powershell
Restart-Service LANShareLite
```

如果根本找不到服务，通常是安装包没有包含 `nssm.exe`，或者安装时没有选择服务组件。此时应重新使用完整安装包，而不是只复制 EXE。

## 修改端口和共享目录

编辑：

```text
C:\ProgramData\lan-share-lite\configs\app.yaml
```

示例：

```yaml
dir: "D:\\LANShare"
port: 8080
bind: ""
enable_upload: true
enable_delete: false
max_upload_size: "10GB"
```

修改后重启服务：

```powershell
Restart-Service LANShareLite
```

`bind: ""` 表示监听所有网卡。不要写成 `127.0.0.1`，否则只有本机能访问。

## 检查防火墙

安装程序会添加名为 `lan-share-lite HTTP` 的入站规则，默认用于 Private/Domain 网络。检查命令：

```powershell
netsh advfirewall firewall show rule name="lan-share-lite HTTP"
Get-NetConnectionProfile
```

家庭或办公室网络建议设为 `Private`。若当前网卡被识别为 Public，即使本机正常，局域网设备也可能被阻止。

## 验证局域网访问

先找出服务端 IP：

```powershell
ipconfig
```

然后在另一台 Windows 电脑运行：

```powershell
Test-NetConnection 192.168.1.100 -Port 8080
```

也可以访问(系统带自测功能)：

```text
http://192.168.1.100:8080/diagnostics
```

## 卸载与数据保留

可以通过“设置 -> 应用”或控制面板卸载。默认卸载程序会移除服务和程序，但保留 `C:\ProgramData\lan-share-lite` 下的用户数据。确认不再需要后再手动删除，避免误伤文件。

下一篇：[macOS、Linux 与 Docker 部署 LAN Share Lite](05-macos-linux-docker.md)

[<< 返回系列目录](../README.md)
