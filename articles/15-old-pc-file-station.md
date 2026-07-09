# 一台旧电脑如何变成免费的局域网文件中转站

闲置电脑不一定要改造成复杂 NAS。只要磁盘健康、网络稳定，它就可以运行 [狼享 LAN Share Lite](https://lan-share.whdatas.com/lite-version)，承担家庭或工作室的局域网文件中转。

## 硬件准备

建议最低条件：

- 64 位 CPU；
- 1GB 以上内存；
- 健康的 SSD 或机械硬盘；
- 千兆网口或稳定 Wi-Fi；
- 可以长期供电并关闭自动睡眠。

文件传输通常更受磁盘和网络影响，不必追求高端 CPU。

## 系统怎么选

### Windows

适合不熟悉 Linux 的用户。安装包可注册 `LANShareLite` 服务，维护直观。注意关闭自动睡眠，并把网络设为 Private。

### Linux

适合长期无人值守，资源占用低。使用 DEB/RPM 和 systemd 管理，数据放在独立磁盘挂载点。

### Docker

适合已有 NAS 或容器环境。要正确映射 `/app/user_data` 和配置目录，并设置自动重启。

## 推荐目录和磁盘规划

```text
/data/lan-share/
├── public/       # 常用共享
├── incoming/     # 临时上传
├── archive/      # 归档，不直接开放删除
└── backup/       # 备份目标，不作为共享根目录
```

不要把备份目录与共享目录混为一谈。Lite 不是备份软件，误删、硬盘损坏和勒索软件仍需要独立备份策略。

## Linux 部署示例

假设数据盘挂载到 `/data/lan-share/public`：

```yaml
dir: "/data/lan-share/public"
port: 8080
bind: ""
enable_upload: true
enable_delete: false
max_upload_size: "20GB"
```

重启并检查：

```bash
sudo systemctl restart lan-share-lite
sudo systemctl status lan-share-lite
```

## 稳定运行检查表

- [ ] 服务开机自动启动；
- [ ] 电脑不会自动睡眠；
- [ ] 数据盘挂载路径固定；
- [ ] 防火墙只允许可信网段；
- [ ] 每周检查磁盘空间和 SMART 状态；
- [ ] 重要数据至少有另一份备份；
- [ ] 用另一台设备测试 `/diagnostics`；
- [ ] 升级前备份配置和用户数据。

## 什么时候旧电脑方案不再够用

当共享成为公司正式基础设施，单台旧电脑可能缺少冗余、集中告警、用户权限、审计和支持保障。此时应该评估更可靠硬件，并使用 [Pro/Enterprise](https://lan-share.whdatas.com/)，而不是不断在免费实例旁边叠加脚本。

下一篇：[免费版什么时候够用，什么时候应该升级 Pro](16-when-to-upgrade.md)

[<< 返回系列目录](../README.md)
