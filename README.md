# uuplugin-dns-patched

[![License: 0BSD](https://img.shields.io/badge/License-0BSD-blue.svg)](LICENSE)

为 UU 加速器 Steam Deck / Linux 插件 (`uuplugin`) 提供 Tailscale DNS 兼容补丁。

无需任何额外常驻守护进程或转发服务，直接修补 UU 内置的 iptables 规则模板，在 DNS 重定向与阻断中排除 Tailscale 地址段，使游戏加速与 MagicDNS / Tailnet 局域网解析完美共存。

---

## 背景与原理

UU 开启加速时会默认插入全局 DNS 劫持规则：
- **IPv4**：将所有出站 DNS 请求重定向（DNAT）至其本地解析服务；
- **IPv6**：直接阻断（DROP）所有出站 DNS 请求。

这会导致 Tailscale 的 MagicDNS 及 Tailnet 内网域名解析全部超时失效。

本项目通过二进制修补（Binary Patch）`uuplugin` 的规则模板字符串：
- **IPv4 DNS DNAT**：规则添加/删除时均排除 `100.64.0.0/10`；
- **IPv6 DNS DROP**：规则添加/删除时均排除 `fd7a:115c:a1e0::/48`。

修补仅限定长调整规则模板，不触碰登录、节点调度等业务逻辑。公共 DNS 仍由 UU 正常加速管理。

---

## 支持版本

目前严格锁定并验证以下版本（二进制输入与输出均校验 SHA256）：

- **UU 版本**：`14.2.2` (Linux x86_64)

| 适用文件 | 类别 | SHA256 |
| --- | --- | --- |
| 官方原始 `uuplugin` | 原版 | `6643c6cde6ff2d38e9817c916adc7df75fa242f00f25befa7353ab041f152482` |
| 官方原版修补后（本包产物） | 补丁后 | `0e82db28ee7a29eb27e729887060fa8c508cfd1010e612e7280828d85b80c357` |
| 旧版 AUR 已安装 `uuplugin`* | 原版 | `28bf89e3b18aed87f7eec759bf5441143c06a0b00306a02cd5184869bf2f9668` |
| 旧版 AUR 文件修补后 | 补丁后 | `3c90b655ab7fcf3f0368b0b0ff7596d9335d522e1da70ea74aaa31fd30d1c248` |

> *\*注：旧版 AUR 安装包多了打包时注入的 debug link，其 `.text` 与 `.rodata` 段与官方原版逐字节一致。*

---

## 安装与使用

### Arch Linux (PKGBUILD)

本项目提供 PKGBUILD（包名 `uuplugin-dns-patched-bin`），并与原版 `uuplugin-bin` 声明冲突与提供关系：

```bash
git clone https://github.com/Koswu/uuplugin-dns-patched.git
cd uuplugin-dns-patched
makepkg -si
```

> **注意：切换或回退服务前**
> 1. 先在 UU 移动端 App 中**停止当前加速**；
> 2. 停止服务：`sudo systemctl stop uuplugin.service`；
> 3. 安装补丁包后，按需启动：`sudo systemctl start uuplugin.service`。

### 独立使用补丁脚本

也可以直接使用 Python 脚本对指定二进制打补丁：

```bash
# 生成补丁文件（目标路径必须不存在）
python3 patch_uuplugin_tailnet.py /path/to/original/uuplugin /path/to/uuplugin-patched

# 运行单元测试
python3 -m unittest -v
```

---

## 相关文档

- [补丁技术细节与实测记录](docs/PATCH.md)
- [AUR 打包与发布规范](docs/AUR.md)
- [构建与兼容性验证记录](docs/BUILD.md)

---

## 许可证与声明

- 本项目自主编写的补丁工具、测试代码及 PKGBUILD 遵循 [0BSD 许可证](LICENSE)。
- `uuplugin` 等二进制文件与商标归网易公司所有。本项目为第三方开源补丁方案，与网易及 Tailscale 官方无任何隶属关系。
