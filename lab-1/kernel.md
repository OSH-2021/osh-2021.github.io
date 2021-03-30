# 编译 Linux 内核

实验用树莓派型号为 raspberrypi 3b+, 并要求编译 64bit 内核.

## 配置交叉编译环境

以 Ubuntu 20.04(x86) 为例 (其他系统和架构请自行安装所需要的环境).

```
sudo apt install git bc bison flex libssl-dev make libc6-dev libncurses5-dev crossbuild-essential-arm64
```

## 下载并解压源码

本次实验采用的[源码地址](https://github.com/raspberrypi/linux/archive/refs/tags/raspberrypi-kernel_1.20210108-1.tar.gz)

由于从 GitHub 直接下载速度缓慢, 可以从[这里](TBA)下载.

解压:

```
tar -xf linux-raspberrypi-kernel_1.20210108-1.tar.gz
mv linux-raspberrypi-kernel_1.20210108-1 linux
```

## 下载其他文件

下载 [tools.tar.gz](tba) 里面包含在 QEMU 以及树莓派上运行所需文件. 解压:

```
tar -xf tools.gz
```

## 编译内核

生成默认配置:

```
cd linux
KERNEL=kernel8 make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcmrpi3_defconfig
```

使用图形界面编辑配置, 如果不熟悉选项, 推荐先编译一次未裁剪的内核:

```
KERNEL=kernel8 make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

编译时加上参数 `-j n`, 根据官方说法, `n` 取你的机器核数*1.5.

```
KERNEL=kernel8 make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image modules dtbs -j n
```

参考时间: 使用 Vlab 平台虚拟机(双核), time 统计时间. 即 45 min 左右.

```
4407.53s user 513.19s system 181% cpu 45:04.30 total
```

## QEMU 运行

*如果想在树莓派中运行, 参考后续章节「在树莓派上运行 Linux 内核和 init (选做)」*

在 QEMU 中运行编译好的内核.

```
qemu-system-aarch64 \
  -kernel linux/arch/arm64/boot/Image \
  -dtb tools/boot_utils/bcm2710-rpi-3-b-plus.dtb \
  -M raspi3 -m 1024 \
  -serial stdio \
  -append "rw loglevel=8"
```

预期现象应是看到 kernel panic.

如果希望看到完整的输出, 可以改参数 `-append "rw earlycon=pl011,0x3f201000 console=ttyAMA0 loglevel=8"` 将它们从串口输出 (即连接到 stdio).

如果不希望看到太多 log, 可以将 loglevel 降低 (最低为 0).

## 运行 RaspiOS (非必做)

尝试在 QEMU 中用自己编译的内核运行 RaspiOS, [镜像下载](https://mirrors.ustc.edu.cn/raspberry-pi-os-images/raspios_lite_arm64/images/raspios_lite_arm64-2020-08-24/2020-08-20-raspios-buster-arm64-lite.zip). <del>推荐用未裁剪内核玩耍.</del>

```
qemu-img resize ../2020-08-20-raspios-buster-arm64-lite.img 2G
qemu-system-aarch64 \
  -kernel linux/arch/arm64/boot/Image \
  -dtb tools/boot_utils/bcm2710-rpi-3-b-plus.dtb \
  -M raspi3 -m 1024 \
  -serial stdio \
  -append "rw loglevel=8 root=/dev/mmcblk0p2 fsck.repair=yes net.ifnames=0 rootwait memtest=1" \
  -drive file=./2020-08-20-raspios-buster-arm64-lite.img,format=raw,if=sd
```

默认用户名 pi 密码 raspberry

Have fun :D

## 参考资料

1. 树莓官方文档: [编译内核](https://www.notion.so/Lab-1-Linux-53d010e1486d4fc180535fcf27ec5c79#e5ba9eda59124591a573a39c334d8fa1) [配置内核](https://www.notion.so/Lab-1-Linux-53d010e1486d4fc180535fcf27ec5c79#8e7ab2b415324a8791e14ba06f457a04)
2. [Debian on QEMU’s Raspberry Pi 3 model](https://translatedcode.wordpress.com/2018/04/25/debian-on-qemus-raspberry-pi-3-model/)