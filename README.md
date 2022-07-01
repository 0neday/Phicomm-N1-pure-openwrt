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

 # select 5.4.170 version from commit
 wget -c https://github.com/unifreq/linux-5.4.y/archive/645cb6197357df079d11c56717165e32d88eef7b.zip
 unzip 645cb6197357df079d11c56717165e32d88eef7b.zip
 mv linux-5.4.y* linux-5.4.y

 # add 5.4.170 config 
 wget -c https://raw.githubusercontent.com/ophub/amlogic-s9xxx-armbian/f5aa923bc7b4dbff6f7ec8b6f20d62a4c77c797e/compile-kernel/tools/config/config-5.4.170
 mv config-5.4.170 linux-5.4.y/.config

 # start recompile kernel
 cd ~/amlogic-s9xxx-armbian
 sudo ./recompile -d -k 5.4.170 -r unifreq -a false
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
mv ~/amlogic-s9xxx-armbian/compile-kernel/output/5.4.170 ~/amlogic-s9xxx-openwrt/amlogic-s9xxx/amlogic-kernel

# make for N1
sudo ~/amlogic-s9xxx-openwrt/make -d -b s905d -k 5.4.170 -a false

```
固件在 `out` 目录下 解压后，用 rufus 写入u盘。

#### 3.写入 eMMC

因为没有用flipy的rootfs，有很多脚本不能使用。只能用原始的方法 [ dd 写入分区](https://github.com/ophub/amlogic-s9xxx-openwrt/issues/185)。

```
dd if=/dev/sda1 of=/dev/mmcblk2p1
dd if=/dev/sda2 of=/dev/mmcblk2p2
sync
```

~~#### 4.遗留问题~~

~~无线蓝牙芯片无法正常驱动，请通过 LAN 连接，地址 192.168.1.1~~

#### 4. 无线驱动
 ```
opkg install kmod-brcmfmac kmod-brcmutil kmod-cfg80211 kmod-mac80211 hostapd-common wpa-cli wpad-basic iw cypress-firmware-43455-sdio	

# copy txt  from flippy firmware
cp *.txt /lib/firmware/brcm/

# 
wget http://ftp.iij.ad.jp/pub/linux/kernel/software/network/wireless-regdb/wireless-regdb-2022.06.06.tar.xz
tar -xzvf *.xz
cp regulatory*  /lib/firmware

reboot 

```

#### Change eth0 using bridge mode 

network
```
config device
        option type 'bridge'
        option name 'br-lan'
        list ports 'eth0'
        option mtu '1500'
        option mtu6 '1500'

config interface 'lan'
        option proto 'static'
        option device 'br-lan'
        option ipaddr '192.168.1.2'
        option netmask '255.255.255.0'
        option gateway '192.168.1.1'
        list dns '127.0.0.1'

```
wireless  add option network 'lan'
```
config wifi-iface 'default_radio0'
        option network 'lan'
```

firewall 

```
config zone
        option name 'lan'
        option input 'ACCEPT'
        option output 'ACCEPT'
        option forward 'ACCEPT'
        list network 'lan'
        list network 'lan6'

```

### License
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.htmlT)


 



