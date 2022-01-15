### 参照 [amlogic-s9xxx-openwrt](https://github.com/ophub/amlogic-s9xxx-openwrt)

改动的地方：内核及模块，根据这个[github里的资源自己编译](https://github.com/ophub/amlogic-s9xxx-openwrt)，
root-default系统用的[openwrt自带的](https://downloads.openwrt.org/releases/21.02.1/targets/armvirt/64/openwrt-21.02.1-armvirt-64-default-rootfs.tar.gz)。

---
#### 1. 编译内核。
[参考这里 amlogic-s9xxx-armbian ](https://github.com/ophub/amlogic-s9xxx-armbian/tree/main/compile-kernel)
##### 1.1 安装依赖
```
sudo apt-get update -y
sudo apt-get full-upgrade -y
sudo apt-get install -y $(curl -fsSL git.io/ubuntu-2004-server)
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

#### 3.已知问题

因为没有用flipy的rootfs，[有很多脚本不能使用。只能用原始的方法 dd 写入分区](https://github.com/ophub/amlogic-s9xxx-openwrt/issues/185)。

```
dd if=/dev/sda1 of=/dev/mmcblk2p1
dd if=/dev/sda2 of=/dev/mmcblk2p2
```
注意：写入后，不能再通过u盘启动其他系统。

解决方法：写入N1 原始 u-boot，未测试，[理论上可以](https://github.com/ophub/amlogic-s9xxx-openwrt/issues/188)
```
dd if=/lib/u-boot/u-boot-2015-phicomm-n1.bin of=/dev/mmcblk2  bs=1 count=444 conv=fsync
dd if=/lib/u-boot/u-boot-2015-phicomm-n1.bin of=/dev/mmcblk2  bs=512 skip=1 seek=1 conv=fsync

```

#### License
[GPLv3](https://www.gnu.org/licenses/gpl-3.0.htmlT)


 


