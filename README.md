

#### 源码来源

[![immortalwrt](https://img.shields.io/badge/immortalwrt-openwrt-orange.svg?style=flat&logo=appveyor)](https://github.com/immortalwrt/immortalwrt)

##### 固件下载链接

- [immortalwrt-25.12固件](https://op.dllkids.xyz/op/firmware/ctc_25.12/)

### 默认插件包含

- openclash
- Nikki
- lucky
- KMS 服务器
- UPNP 自动端口转发
- 默认多个主题
- 默认管理 IP: 192.168.1.251, 用户名 root，密码 password
- 修改默认ip

### 2026-07-30 编译失败排查记录

最近两次手动触发的 `immortalwrt_x86_R25_12` 和 `immortalwrt_R2s_R25_12` 均在 `🧩 编译固件` 阶段失败：

- x86 run: <https://github.com/yelc66/openwrt_Build/actions/runs/30526546236>
- R2s run: <https://github.com/yelc66/openwrt_Build/actions/runs/30526546221>

失败点一致，均为 `package/dae/daed`：

```text
ERROR: package/dae/daed failed to build.
make[3]: *** [Makefile:190: .../daed-2026.07.17/wing/.built] Error 1
../../webrender/webrender.go:23:12: pattern web: cannot embed directory web: contains no embeddable files
```

当前 workflow 使用 `immortalwrt/immortalwrt` 的 `openwrt-25.12` 分支，并额外克隆 `QiuSimons/luci-app-daed` 的 `kix` 分支到 `package/dae`。该分支最新提交为 `9511862b8cf8`，时间为 `2026-07-17T09:56:18Z`，与失败日志中的 `daed-2026.07.17` 对应。继续检查 `daeuniverse/daed` 本体仓库后，确认默认分支是 `main`，当前 `main` 仍是失败使用的 `671e65d2fdcd62fe6a3ec18ecda209c5addea898`，没有比本次失败版本更新的修复提交。

`QiuSimons/luci-app-daed` 的 `daed/Makefile` 已包含预期的前端构建流程：下载 Node.js、安装 `pnpm`、执行 `pnpm install` 和 `pnpm build --filter daed`，再把 `apps/web/dist/*` 复制到 `wing/webrender/web`。本次报错说明 Go embed 编译时该目录为空，实际问题更可能出在前端构建产物没有生成或单线程重试没有重新执行 `Build/Prepare`，而不是 `daeuniverse/daed` 本体已有未同步的新修复。

同仓库里的 `luci-app-daed` 目录是 LuCI 前端管理包，`Makefile` 仅声明 `LUCI_DEPENDS:=+daed +zoneinfo-asia +luci-compat`，不会参与 `daed` 二进制的 `wing/webrender/web` embed 构建。该目录最近相关提交停在 `2026-06-20`，主要是 `luci_daed` init/hotplug/DNS 处理调整，不是本次 `package/dae/daed` 编译失败的修复来源。

原 fork 仓库 `kenzok8/openwrt_Build` 的主分支最新提交为 `31573ed5e1da`，时间为 `2026-07-23T08:50:14Z`，提交信息是 `Fix Lienol BTF build`，内容是为 Lienol 构建补充 `pahole`，与本次 `daed` 的 `web` embed 编译错误无直接关系。不过主分支还包含多项 workflow 维护更新，例如 25.12 workflow 冲突清理、GitHub Actions 版本更新、固件上传路径迁移等。

推荐处理方式：

- 如果当前固件不需要 `daed`，可先从 `immortalwrt_r25.config` 和 `immortalwrt_r2s.config` 中取消 `CONFIG_PACKAGE_daed`、`CONFIG_PACKAGE_luci-app-daed`。
- 如果需要保留 `daed`，优先检查 `package/dae/daed` 的 `Build/Prepare` 阶段是否成功生成 `apps/web/dist` 并复制到 `wing/webrender/web`。必要时清理 `openwrt/build_dir/target-*/daed-*` 后重编，或在 workflow 中对 `daed` 单独执行 clean/compile 以避免使用缺失前端产物的旧构建目录。
- 推荐合并 `kenzok8/openwrt_Build` 主分支更新，但需要通过单独分支或 PR 合并并人工处理冲突。当前本仓库与上游主分支已经明显分叉：上游删除了 `immortalwrt_R25.yml`、`immortalwrt_R2s.yml`、`immortalwrt_r25.config`、`immortalwrt_r2s.config`，并改为多套 workflow。因此不建议直接覆盖式合并，应先确认是否保留现有 R25/R2s 构建入口。

### Debian/Ubuntu 编译环境配置

对于 Debian/Ubuntu 用户，有两种方式安装编译依赖：

#### 方法 1：手动通过 APT 安装依赖

```bash
sudo apt update -y
sudo apt full-upgrade -y
sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
  bzip2 ccache clang cmake cpio curl device-tree-compiler ecj fastjar flex gawk gettext gcc-multilib \
  g++-multilib git gnutls-dev gperf haveged help2man intltool lib32gcc-s1 libc6-dev-i386 libelf-dev \
  libglib2.0-dev libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev libncurses-dev libpython3-dev \
  libreadline-dev libssl-dev libtool libyaml-dev libz-dev lld llvm lrzsz mkisofs msmtp nano \
  ninja-build p7zip p7zip-full patch pkgconf python3 python3-pip python3-ply python3-docutils \
  python3-pyelftools qemu-utils re2c rsync scons squashfs-tools subversion swig texinfo uglifyjs \
  upx-ucl unzip vim wget xmlto xxd zlib1g-dev zstd
```

#### 方法 2：使用自动化脚本

```bash
sudo bash -c 'bash <(curl -s https://build-scripts.immortalwrt.org/init_build_environment.sh)'
```

### r2s 包含openlist

```bash
sed -i 's/192.168.1.1/192.168.2.1/g' package/base-files/files/bin/config_generate
```

- 替换终端为bash

```bash
sed -i 's/\/bin\/ash/\/bin\/bash/' package/base-files/files/etc/passwd
```

- 添加新的主题

```bash
git clone https://github.com/kenzok8/luci-theme-ifit.git package/lean/luci-theme-ifit
```

- 添加常用软件包

```bash
git clone https://github.com/kenzok8/openwrt-packages.git package/openwrt-packages
```

- 删除默认密码

```bash
sed -i "/CYXluq4wUazHjmCDBCqXF/d" package/lean/default-settings/files/zzz-default-settings
```

- 取消bootstrap为默认主题

```bash
sed -i 's/luci-theme-bootstrap/luci-theme-argon/g' feeds/luci/collections/luci/Makefile
```
