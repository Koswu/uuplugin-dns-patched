# 仓库提取后的构建验证（2026-10-28）

与 PATCH.md 中历史 App 加速实验分开记录。

- 新建一次性 Arch Linux 虚拟机；没有挂载宿主目录，没有安装或运行 UU。
- 安装 base-devel、Python、namcap。以普通用户执行 makepkg，完整下载源校验、prepare、check、package 通过。
- 7 项单元测试通过，PKGBUILD 的 bash 语法检查通过，`.SRCINFO` 与 makepkg 自动生成结果一致。
- 网易原始文件和此前实测安装文件的 `.text`、`.rodata` 完全一致。相同补丁位置分别执行完整输入、输出摘要校验。
- 成品包 `/usr/bin/uuplugin` SHA256 为 `0e82db28ee7a29eb27e729887060fa8c508cfd1010e612e7280828d85b80c357`，大小 5,210,480 字节。
- 通过 AUR RPC 查询 uuplugin，仅有原版 `uuplugin-bin`；查询时未发现计划包名被占用。发布时仍需复查。

## namcap 结果与未完成项

PKGBUILD 检查无输出。成品包检查仍有一项错误，不能称为全部通过：

> Uncommon license identifiers such as '0BSD, LicenseRef-UU' require license files below /usr/share/licenses/uuplugin-dns-patched-bin/. Found 1/2 required license files.

仓库只持有自己编写部分的 0BSD 文本。网易官方压缩包未附许可证，不用虚构许可证或无关 NOTICE 冒充。AUR 正式提交前需获得适用的上游许可文本、确认该发布方式符合其条款，再将其作为校验过的 source 安装到许可证目录。

另有 9 项警告：uuplugin executable stack；两份上游 ELF 均缺少 FULL RELRO、PIE；iproute2、iptables、kmod、systemd 被提示可能不需要。前 5 项来自未重新编译的上游 ELF，不为消除警告而改写未知二进制；后 4 项依赖用于程序运行时调用外部命令和服务，保留声明。

独立只读代码审查未发现阻碍 GitHub 源码公开的问题。没有在本次仓库提取后重新进行 App 加速测试，也没有修改宿主 UU、DNS、Tailscale 配置。
