# AUR 发布准备

目标包：`uuplugin-dns-patched-bin`；GitHub 仓库：`Koswu/uuplugin-dns-patched`。

## 采用的规范

- [AUR submission guidelines](https://wiki.archlinux.org/title/AUR_submission_guidelines)：先检查重复包；带额外补丁的变体采用独立名字，通过 conflicts/provides 声明关系；不用 replaces 强迫原包用户迁移。
- [Arch package guidelines](https://wiki.archlinux.org/title/Arch_package_guidelines)：配置放 `/etc`，服务放 `/usr/lib/systemd/system`，自行编写的打包代码采用 0BSD。
- [PKGBUILD(5)](https://man.archlinux.org/man/PKGBUILD.5.en.html)：prepare 中打补丁、check 中测试、package 仅安装到 pkgdir；Python 为构建依赖，网络工具为运行依赖。
- [非自由软件打包](https://wiki.archlinux.org/title/Nonfree_applications_package_guidelines)：上游程序没有开源，构建时从官方固定 URL 下载。仓库及 AUR 均不提交二进制。

PKGBUILD 启用配置备份、关闭 strip/debug，校验官方压缩包及全部本地 source 文件。补丁还独立校验输入与输出二进制。上游未随下载包附许可证，`LicenseRef-UU` 只是标识而非授权；LICENSE.patch 只覆盖本项目代码。正式提交前需核实上游条款，若禁止修改或该下载方式则不能据此配方发布。

## 检查

```sh
python3 -m unittest -v
bash -n PKGBUILD
makepkg --verifysource
makepkg --cleanbuild --force
namcap PKGBUILD
namcap ./uuplugin-dns-patched-bin-14.2.2-1-x86_64.pkg.tar.zst
makepkg --printsrcinfo > .SRCINFO
```

建议在干净 Arch chroot 或隔离虚拟机执行构建，确认声明依赖齐全。构建本身不执行 UU；行为测试需要独立虚拟机和真人在 UU App 启停。

```sh
bsdtar -xOf uuplugin-dns-patched-bin-14.2.2-1-x86_64.pkg.tar.zst usr/bin/uuplugin | sha256sum
# 应为 README 中 patched SHA256
```

修改本地 source 后同步修改 PKGBUILD 中的 sha256sums，再生成 `.SRCINFO`；包行为变更递增 pkgrel，上游版本变化递增 pkgver 并重置 pkgrel。

## 提交

先确认 AUR 账号、SSH 公钥和包名是否可用，并完成上游许可核对。以下是人工发布命令，本仓库初始化不会自动提交 AUR：

```sh
git clone ssh://aur@aur.archlinux.org/uuplugin-dns-patched-bin.git /tmp/uuplugin-dns-patched-aur
cp PKGBUILD .SRCINFO LICENSE uuplugin.service patch_uuplugin_tailnet.py test_uuplugin_patch.py /tmp/uuplugin-dns-patched-aur/
cd /tmp/uuplugin-dns-patched-aur
# 检查任何已存在的历史，勿覆盖已有维护者工作
makepkg --printsrcinfo > .SRCINFO
git add PKGBUILD .SRCINFO LICENSE uuplugin.service patch_uuplugin_tailnet.py test_uuplugin_patch.py
git diff --cached --check
git diff --cached
git commit -m 'Initial package: UU 14.2.2 with Tailnet DNS patch'
git push origin master
```

使用单独的 AUR checkout，使 master 历史只包含打包所需文件。不要推送下载压缩包、src、pkg、构建产物、实测设备信息或整个基础设施仓库。
