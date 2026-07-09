# 第一次使用 LAN Share Lite：共享目录、二维码与访问地址

安装完成后，很多人第一反应是：“页面打开了，然后呢？”

[LANShare Lite](https://lan-share.whdatas.com/lite-version) 的首次使用可以拆成四件事：确认共享目录、选择访问地址、完成一次上传下载、设置安全开关。

## 1. 确认共享目录

浏览器打开：

```text
http://localhost:8080
```

页面展示的是 `app.yaml` 中 `dir` 指向的目录。默认安装路径因系统不同：

| 系统 | 默认数据目录 |
|---|---|
| Windows | `C:\ProgramData\lan-share-lite\user_data` |
| macOS | `/Library/Application Support/lan-share-lite/user_data` |
| Linux | `/var/lib/lan-share-lite/user_data` |
| Docker | `/app/user_data` |

要改到其他磁盘，修改配置：

```yaml
dir: "D:\\TeamShare"
```

或 Linux/macOS：

```yaml
dir: "/data/team-share"
```

## 2. 选择正确访问地址

`localhost` 只能在服务端自己使用。给其他设备访问时，应使用页面显示的局域网 IP，例如：

```text
http://192.168.1.100:8080
```

页面二维码编码的也是访问地址，手机扫描后即可打开。mDNS 可用时还能使用：

```text
http://lan-share-lite.local:8080
```

如果设备有 Wi-Fi、有线网卡、虚拟机和 VPN，页面可能显示多个 IP。通常选择与客户端处于同一网段的地址。

## 3. 完成第一次上传和下载

在客户端页面：

1. 点击上传按钮选择文件；
2. 或把文件直接拖到上传区域；
3. 上传整个目录时使用“上传目录”；
4. 点击文件右侧下载按钮验证下载；
5. 使用搜索框确认文件可被找到。

同名文件上传前会进行冲突检查，避免无提示覆盖。先用一个无关紧要的小文件完成测试，再处理正式资料。

## 4. 配置上传和删除权限

Lite 没有用户账号，因此开关对所有能访问页面的人生效：

```yaml
enable_upload: true
enable_delete: false
```

只做资料分发时关闭上传；允许团队提交文件但不允许误删时保持删除关闭。修改后重启服务。

## 5. 看懂连接质量

页面可以检测浏览器到服务端的延迟、抖动以及上传/下载吞吐估算。它不是运营商测速工具，而是判断“当前连接是否适合传大文件”的体验指标。

当页面提示上传方向较慢时，可以：

- 靠近 Wi-Fi 路由器；
- 改用 5GHz/6GHz 或有线网络；
- 暂停其他占带宽任务；
- 避免同时上传多个大文件。

## 第一次使用检查表

- [ ] 数据目录位于预期磁盘；
- [ ] 另一台设备能打开 IP 地址；
- [ ] 上传和下载测试成功；
- [ ] 删除功能按需关闭；
- [ ] 记住 `/diagnostics` 排障入口；
- [ ] 敏感文件没有放在公共 Wi-Fi 中共享。

下一篇：[localhost 能打开，其他电脑打不开？](07-network-diagnostics.md)

[<< 返回系列目录](../README.md)
