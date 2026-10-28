# 构建验证记录

## 隔离环境打包验证

- **系统环境**：Arch Linux (Incus 虚拟机环境，无宿主系统依赖)
- **依赖安装**：`base-devel`、`python`、`namcap`
- **构建结果**：
  - 源码下载、校验、补丁注入、单元测试、打包全流程一次通过。
  - 补丁后二进制 SHA256 为 `0e82db28ee7a29eb27e729887060fa8c508cfd1010e612e7280828d85b80c357`，文件大小 5,210,480 字节。
  - `.SRCINFO` 与 `makepkg --printsrcinfo` 完全一致。

---

## namcap 检查说明

- **PKGBUILD**：检查通过无告警。
- **构建包警告说明**：
  - 上游闭源 ELF 文件缺少 PIE / RELRO 标志，属上游编译特性，补丁保持原样不额外注入未知改动。
  - `iproute2` / `iptables` / `systemd` 被提示可能不需要：实际由 `uuplugin` 在运行时以外部子进程形式调用，因此保留作为明确依赖。
