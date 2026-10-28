# uuplugin-dns-patched

为 UU Steam Deck Linux 插件添加 Tailscale DNS 兼容补丁，无需额外常驻监听服务。

UU 在启动加速时会插入 IPv4 DNS 重定向及 IPv6 DNS 阻断规则，可能使 Tailscale DNS 无响应，进而影响普通网页和内网域名解析。本工具仅在构建副本中修改这些规则：

- IPv4 DNS DNAT 的添加、删除均排除 `100.64.0.0/10`。
- IPv6 TCP/UDP DNS DROP 的添加、删除均排除 `fd7a:115c:a1e0::/48`。

补丁不更改系统 DNS、Tailscale 设置或路由，不包含额外守护进程。只有安装后运行 UU 时，UU 才会按修改后的逻辑管理规则。公共 DNS 仍由 UU 原逻辑处理。

## 支持范围

仅支持 **UU 14.2.2 / Linux x86_64** 的两种固定文件档案（官方原始文件及先前 AUR 安装文件）。完整输入、输出 SHA256 都必须匹配；版本号相同但内容不同也会拒绝。不要只修改版本号或绕过校验来支持新版。

| 文件 | SHA256 |
| --- | --- |
| 网易官方 `uu.tar.gz` | `6881c8bd388630edfdbceef1f723bf43cf75b0ce5122ee891e99d2fb86c96652` |
| 官方原始 `uuplugin` | `6643c6cde6ff2d38e9817c916adc7df75fa242f00f25befa7353ab041f152482` |
| 官方文件打补丁后（本包安装值） | `0e82db28ee7a29eb27e729887060fa8c508cfd1010e612e7280828d85b80c357` |
| AUR 安装原始 `uuplugin` | `28bf89e3b18aed87f7eec759bf5441143c06a0b00306a02cd5184869bf2f9668` |
| AUR 安装文件打补丁后 | `3c90b655ab7fcf3f0368b0b0ff7596d9335d522e1da70ea74aaa31fd30d1c248` |

这是非官方补丁，与网易及 Tailscale 无隶属关系。只验证了 Steam 商店加速；不保证所有游戏、网络环境或更新后的 UU 行为兼容。

## Arch Linux 构建与安装

计划使用的 AUR 包名：`uuplugin-dns-patched-bin`。仓库已提供 PKGBUILD 和 `.SRCINFO`，不代表已经发布到 AUR。

```sh
git clone https://github.com/Koswu/uuplugin-dns-patched.git
cd uuplugin-dns-patched
makepkg -s
```

构建会从网易下载固定版本，校验后离线打补丁并运行测试。`makepkg -s` 可能安装构建依赖，但不会安装或启动 UU。检查 PKGBUILD 与生成的包后再安装：

```sh
sudo pacman -U ./uuplugin-dns-patched-bin-14.2.2-1-x86_64.pkg.tar.zst
```

此包提供 `uuplugin-bin`，与 `uuplugin-bin`、`uuplugin` 互斥。`/etc/uuplugin/uu.conf` 使用 pacman 的配置备份机制；包不会自动启动、启用或重启服务，也不修改 Tailscale 偏好。

从原版切换前，应先在 UU App 停止加速，再停止服务，避免旧规则遗留：

```sh
sudo systemctl stop uuplugin.service
# 安装补丁包后，按需启动
sudo systemctl start uuplugin.service
```

在 App 中重新加速后，验证普通网页、Steam 以及自己的 Tailnet 内网域名。需要使用 Tailscale DNS 时自行启用 `tailscale set --accept-dns=true`；此设置是否适合你的网络应单独确认。

回退时同样先停止加速和服务，再安装原版 `uuplugin-bin`。不使用无差别清空 iptables 的方式恢复网络。

## 单独生成补丁副本

```sh
python3 patch_uuplugin_tailnet.py /path/to/original/uuplugin /tmp/uuplugin-patched
python3 -m unittest -v
```

目标必须不存在。工具拒绝覆盖源文件、已有目标、目标符号链接或硬链接；不需要 root，不修改已安装的程序。

## 验证与维护

隔离虚拟机中的真实 UU App 启停验证了规则清理、重建，以及 Tailscale DNS 和 Steam 流量路径。94 项 DNS 检查通过；1776 次长采样查询出现两次孤立的 1 秒超时，随后恢复，因此不承诺零中断。[实测与补丁说明](docs/PATCH.md)

[发布 AUR 的步骤和检查项](docs/AUR.md)，[本次构建结果及 namcap 未通过项](docs/BUILD.md)。本仓库不上传官方或修改后的 UU 二进制；AUR 构建时从网易下载原版。

## 许可证

本项目自行编写的补丁工具、测试及打包文件使用 [0BSD](LICENSE)。网易的 `uuplugin`、`xuplugin-guardian` 和配置仍受其自身条款约束，0BSD 不适用于这些文件。官方压缩包未附带许可证文本；**公开下载不等于允许修改或再分发**。本项目尚未确认上游条款对该补丁发布方式的许可，AUR 正式提交前须核对，不宣称已满足上游授权要求。
