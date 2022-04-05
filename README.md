### 参照 [amlogic-s9xxx-openwrt](https://github.com/ophub/amlogic-s9xxx-openwrt)

改动的地方：内核及模块，根据这个github里的[资源自己编译](https://github.com/ophub/amlogic-s9xxx-openwrt)，
root-default系统用的[openwrt自带的](https://downloads.openwrt.org/releases/21.02.1/targets/armvirt/64/openwrt-21.02.1-armvirt-64-default-rootfs.tar.gz)。

---
#### 1. 编译内核。
[参考这里 amlogic-s9xxx-armbian ](https://github.com/ophub/amlogic-s9xxx-armbian/tree/main/compile-kernel)
##### 1.1 安装依赖
```
sudo apt-get update -y
sudo apt-get full-upgrade -y
sudo apt-get install -y  build-essential qemu-user-static gcc-aarch64-linux-gnu gcc-10-aarch64-linux-gnu pkg-config netfilter-persistent xz-utils rename tar openssl libidn11-dev libncurses-dev minizip bc asciidoc binutils bzip2 gawk ninja-build u-boot-tools gettext git fakeroot libncurses5-dev libz-dev patch python3 python2.7 pigz zstd zip unzip zlib1g-dev btrfs-progs bison uuid-runtime mount parted util-linux dosfstools subversion flex uglifyjs libpixman-1-dev git-core gzip p7zip p7zip-full msmtp libssl-dev vim busybox texinfo libglib2.0-dev xmlto qemu-utils qemu qemu-system qemu-user libidn11 initramfs-tools gdb-multiarch upx libelf-dev autoconf automake libtool autopoint device-tree-compiler antlr3 gperf wget curl swig rsync 
```
##### 1.2 git kernel source and compile kernel initrd and kernel modules etc
```
 cd ~
 git clone --depth 1 https://github.com/ophub/amlogic-s9xxx-armbian.git 
 mkdir -p amlogic-s9xxx-armbian/compile-kernel/kernel/
 cd amlogic-s9xxx-armbian/compile-kernel/kernel/

 # select 5.4.180 version from commit
 wget -c https://github.com/unifreq/linux-5.4.y/archive/16ddf6290d54418345a954e2d0d99089a4743997.zip
 unzip 16ddf6290d54418345a954e2d0d99089a4743997.zip
 mv linux-5.4.y* linux-5.4.y

 # add 5.4.180 config 
 wget -c https://raw.githubusercontent.com/0118Add/amlogic-s9xxx-armbian/main/compile-kernel/tools/config/config-5.4.180
 mv config-5.4.180 linux-5.4.y/.config

 # start recompile kernel
 cd ~/amlogic-s9xxx-armbian
 sudo ./recompile -d -k 5.4.180 -r unifreq -a false
```
after that get all zip file from `compile-kernel/output`


#### 2.构建 openwrt 固件。
```
cd ~
git clone --depth 1 https://github.com/ophub/amlogic-s9xxx-openwrt.git

# get openwrt offical default-rootfs 
mkdir -p ~/amlogic-s9xxx-openwrt/openwrt-armvirt
cd ~/amlogic-s9xxx-openwrt/openwrt-armvirt 
wget -c https://downloads.openwrt.org/releases/21.02.1/targets/armvirt/64/openwrt-21.02.1-armvirt-64-default-rootfs.tar.gz

# mv compiled kernel into 
mkdir -p ~/amlogic-s9xxx-openwrt/amlogic-s9xxx/amlogic-kernel
mv ~/amlogic-s9xxx-armbian/compile-kernel/output/5.4.180 ~/amlogic-s9xxx-openwrt/amlogic-s9xxx/amlogic-kernel

# make for N1
sudo ~/amlogic-s9xxx-openwrt/make -d -b s905d -k 5.4.180 -a false

```
固件在 `out` 目录下 解压后，用 rufus 写入u盘。

#### 3.写入 eMMC

因为没有用flipy的rootfs，有很多脚本不能使用。只能用原始的方法 [ dd 写入分区](https://github.com/ophub/amlogic-s9xxx-openwrt/issues/185)。

```
dd if=/dev/sda1 of=/dev/mmcblk2p1
dd if=/dev/sda2 of=/dev/mmcblk2p2
```
注意：写入后，不能再通过u盘启动其他系统。

#### 4.问题及解决方法

解决方法：
Change bootargs LABEL=ROOTFS to [ UUID={UUID}](https://github.com/ophub/amlogic-s9xxx-openwrt/issues/189#issuecomment-1013812210) in `uEnv.txt`

### License
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.htmlT)


 



