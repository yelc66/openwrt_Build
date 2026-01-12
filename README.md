

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
