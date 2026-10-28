# uuplugin-dns-patched

[![License: 0BSD](https://img.shields.io/badge/License-0BSD-blue.svg)](LICENSE)

针对网易 UU 加速器 Linux / Steam Deck 插件 (`uuplugin`) 的通用二进制补丁，解决开启加速后 Tailscale DNS（MagicDNS 及局域网内网解析）失效的问题。

适用于所有运行 UU 插件的 Linux 环境（SteamOS / Steam Deck、Ubuntu、Debian、Fedora、Arch Linux 等）。

---

## 问题背景与原理

UU 在开启加速时会默认插入全局 iptables 规则：
- **IPv4**：将所有出站 DNS 请求重定向（DNAT）至其本地解析服务；
- **IPv6**：直接阻断（DROP）所有出站 DNS 请求。

这会导致 Tailscale 的 MagicDNS 及 Tailnet 内网域名解析全部超时失效。

本项目通过直接修补 `uuplugin` 可执行文件中的规则模板字符串（二进制定长 Patch）：
- **IPv4 DNS DNAT**：添加与清理时均排除 `100.64.0.0/10`；
- **IPv6 DNS DROP**：添加与清理时均排除 `fd7a:115c:a1e0::/48`。

**特点**：
- **无常驻进程**：无需后台守护进程或外部代理，直接由 UU 原生逻辑管理。
- **纯粹且安全**：只修补规则模板中的排除条件，不触碰登录、节点调度等业务逻辑。
- **严格校验**：修补前后均校验 SHA256，防止错误修改未知版本的二进制。

---

## 支持版本

目前严格支持并校验以下版本：

- **上游版本**：`14.2.2` (Linux x86_64)

| 适用文件 | 类别 | SHA256 |
| --- | --- | --- |
| 官方原始 `uuplugin` | 原版 | `6643c6cde6ff2d38e9817c916adc7df75fa242f00f25befa7353ab041f152482` |
| 官方文件修补后 | 补丁后 | `0e82db28ee7a29eb27e729887060fa8c508cfd1010e612e7280828d85b80c357` |
| 携带 debuglink 的 `uuplugin`* | 原版 | `28bf89e3b18aed87f7eec759bf5441143c06a0b00306a02cd5184869bf2f9668` |
| 携带 debuglink 的修补后 | 补丁后 | `3c90b655ab7fcf3f0368b0b0ff7596d9335d522e1da70ea74aaa31fd30d1c248` |

> *\*注：部分打包产物可能带有构建时注入的 ELF debug link，其 `.text` 与 `.rodata` 核心段与官方版完全一致。*

---

## 通用使用方法（发行版无关）

本工具仅依赖 Python 3 标准库，无需额外依赖。

### 1. 下载工具

```bash
git clone https://github.com/Koswu/uuplugin-dns-patched.git
cd uuplugin-dns-patched
```

可运行测试验证补丁逻辑：
```bash
python3 -m unittest -v
```

### 2. 生成补丁副本

指定原版 `uuplugin` 路径与生成的目标路径（目标文件必须不存在，避免误覆盖）：

```bash
python3 patch_uuplugin_tailnet.py /path/to/original/uuplugin /path/to/uuplugin-patched
```

脚本将输出修补前后的校验和与文件大小，并赋予可执行权限。

### 3. 应用并生效

无论使用 systemd 还是手动启动 UU：

1. **先停止加速**：在 UU 手机 App 中点击停止当前加速；
2. **停止服务**：停止正在运行的 UU 进程（如 `sudo systemctl stop uuplugin`）；
3. **替换二进制**：备份原程序，并将生成的 `uuplugin-patched` 覆盖到实际安装位置（如 `/usr/bin/uuplugin`）；
4. **重新启动**：启动服务并在 App 中重新开启加速。

---

## 发行版打包支持

### Arch Linux / SteamOS (AUR)

对于 Arch Linux 或开启了可写模式的 Steam Deck 用户，本项目已提供开箱即用的 AUR 包：

- **AUR 软件包**：[`uuplugin-dns-patched-bin`](https://aur.archlinux.org/packages/uuplugin-dns-patched-bin)

```bash
yay -S uuplugin-dns-patched-bin
# 或
paru -S uuplugin-dns-patched-bin
```

该包构建时自动从官方 CDN 拉取原始文件并离线打补丁，与原版 `uuplugin-bin` 互斥。

Arch 打包配方及指南位于 [packaging/arch/](packaging/arch/)。

---

## 更多文档

- [补丁技术细节与实测测试矩阵](docs/PATCH.md)
- [Arch / AUR 打包与发布规范](packaging/arch/AUR.md)

---

## 许可证与声明

- 本项目编写的补丁脚本、测试及打包定义均遵循 [0BSD 许可证](LICENSE)。
- `uuplugin` 等二进制文件及品牌商标版权归网易公司所有。本项目为第三方开源补丁方案，与网易及 Tailscale 官方无任何隶属关系。
