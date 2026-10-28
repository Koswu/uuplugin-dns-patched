# AUR 发布与维护指南

- **包名**：`uuplugin-dns-patched-bin`
- **上游源码**：[Koswu/uuplugin-dns-patched](https://github.com/Koswu/uuplugin-dns-patched)

---

## 打包规范与设计

1. **命名与关系**：遵循 [AUR Submission Guidelines](https://wiki.archlinux.org/title/AUR_submission_guidelines)，采用独立变体包名，并通过 `provides=("uuplugin-bin=${pkgver}")` 和 `conflicts=('uuplugin-bin')` 明确互斥与替代关系。
2. **下载与打包**：根据 [非自由软件打包规范](https://wiki.archlinux.org/title/Nonfree_applications_package_guidelines)，仓库与 AUR 均不托管二进制，构建阶段直接从官方 CDN 获取并做 SHA256 校验。
3. **打包参数**：
   - 使用 `options=('!strip' '!debug')`，防止 pacman 打包时破坏校验过的补丁二进制。
   - 包含 `/etc/uuplugin/uu.conf` 配置备份声明。

---

## 本地构建与检查

在提交前，建议在干净环境执行以下常规检查：

```bash
# 1. 运行补丁自测
python3 -m unittest -v

# 2. 语法与源码校验
bash -n PKGBUILD
makepkg --verifysource

# 3. 本地构建与打包检查
makepkg --cleanbuild --force
namcap PKGBUILD
namcap ./uuplugin-dns-patched-bin-*-x86_64.pkg.tar.zst

# 4. 更新 .SRCINFO
makepkg --printsrcinfo > .SRCINFO
```

---

## 提交到 AUR

初次发布或更新至 AUR：

```bash
# 检出 AUR 仓库
git clone ssh://aur@aur.archlinux.org/uuplugin-dns-patched-bin.git /tmp/aur-pkg
cd /tmp/aur-pkg

# 拷贝发布文件（仅提交 PKGBUILD、.SRCINFO、补丁脚本、服务文件及许可证）
cp /path/to/uuplugin-dns-patched/{PKGBUILD,.SRCINFO,LICENSE,uuplugin.service,patch_uuplugin_tailnet.py,test_uuplugin_patch.py} .

# 检查并推送
git add PKGBUILD .SRCINFO LICENSE uuplugin.service patch_uuplugin_tailnet.py test_uuplugin_patch.py
git commit -m "chore: release 14.2.2-1"
git push origin master
```
