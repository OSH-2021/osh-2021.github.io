# 构建初始内存盘

## 一个简单的例子

`init.c`:

```c
#include <stdio.h>

int main() {
    printf("TaokyStrong!\n");
    return 0;
}
```

静态链接编译:

```
aarch64-linux-gnu-gcc -static init.c -o init
```

创建一个新的目录用于放置文件。在新的目录下打包 initrd：

```
find . | cpio --quiet -H newc -o | gzip -9 -n > ../initrd.cpio.gz
```

在 QEMU 中运行:

```
qemu-system-aarch64 \
  -kernel linux/arch/arm64/boot/Image \
  -initrd initrd.cpio.gz \
  -dtb tools/boot_utils/bcm2710-rpi-3-b-plus.dtb \
  -M raspi3 -m 1024 \
  -serial stdio \
  -append "rw loglevel=8"
```

## 创建设备文件

以 `/dev/fb0` 为例: 作为一个设备文件，对 `/dev/fb0` 文件的读写等操作实际上是在操作 framebuffer, 在硬件设备上的反映就是屏幕的图像输出等. 
串口设备 `/dev/ttyS0` 同理.

在 Linux 中主要有两类设备文件：块设备文件（有缓冲）、字符设备文件（无缓冲）。

块设备 (b)：以块为单位与硬件设备传输文件。对它的写入操作会被缓存，由内核在合适的时候发送给硬件。
字符设备 (c)：以字节为单位与硬件设备传输文件。

设备文件需要使用 `mknod` 创建。在创建时，还需要提供主设备号 (Major) 和次设备号 (Minor)。设备文件一般位于 `/dev/`，可以使用 `ls -l` 查看它们的信息。

```
$ ls -l /dev/null
crw-rw-rw- 1 root root 1, 3 Feb 14 20:27 /dev/null
$ ls -l /dev/sda
brw-rw---- 1 root disk 8, 0 Feb 14 20:27 /dev/sda
```

这里可以看到，空设备 (`/dev/null`) 为字符设备 (c)，主设备号为 1，次设备号为 3。第一块硬盘对应的设备 (`/dev/sda`) 为块设备 (b)，主设备号为 8，次设备号为 0。

你需要在执行测试程序之前先用 `mknod` 创建对应设备文件. 已知 `/dev/ttyS0` 为字符设备文件，主设备号为 4，次设备号为 64；`/dev/ttyAMA0` 为字符设备文件，主设备号为 204，次设备号为 64；`/dev/fb0` 为字符设备文件，主设备号为 29，次设备号为 0。

你可以使用 `man 1 mknod` 查看 `mknod` 程序的帮助，使用 `man 2 mknod` 查看 `mknod` 系统调用的帮助。这两者都需要最高用户权限。

以下是一个在 C 语言中使用 `mknod()` 系统调用，在当前目录创建空设备文件 (null) 的示例：

```
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/sysmacros.h>

int main() {
    if (mknod("./null", S_IFCHR | S_IRUSR | S_IWUSR, makedev(1, 3)) == -1) {
        perror("mknod() failed");
    }
    return 0;
}
```

```
$ gcc mknod_example.c -o mknod_example
$ sudo ./mknod_example
$ ls -l null
crw------- 1 root root 1, 3 Mar  1 17:10 null
```

你可以选择:

1. 在 `init` 执行过程中创建设备文件. (推荐)
2. 先使用 mknod 创建设备文件，然后再打包 initrd (不推荐). 如果一定想这么做请注意不要写入错误的主次设备和避免硬件损坏. 如果在 Vlab 平台上并不能直接 mknod, 需要使用 fakeroot 环境, [参考操作](https://osh-2020.github.io/lab-1/initrd/#fakeroot-initrd).

## 关于测试程序

在提供的 `tools.tar.gz` 中的 binary 文件夹下有 3 个测试程序.

- 程序 1: 调用 Linux 下的系统调用, 每隔 1 秒显示一行信息, 共显示 4 条消息.
- 程序 2: 向串口输出一条信息。此程序依赖 `/dev/ttyS0` 或 `/dev/ttyAMA0` 设备文件。(具体看[树莓派官方对于串口的说明](https://www.raspberrypi.org/documentation/configuration/uart.md))
- 程序 3: 在 TTY 中使用 Framebuffer 显示一张图片。此程序依赖 `/dev/fb0` 设备文件。

你需要做的是：编写一个 init 程序，执行这三个程序。具体的要求见介绍页面。

## 试试 BusyBox (非必做, 但是推荐玩一玩)

BusyBox 是一个将许多常用 Unix 命令行工具打包进一个二进制文件的项目，在嵌入式系统等存储空间有限的环境中非常常见。你可以选择使用 BusyBox 提供的工具来编写你的 init 程序，并打包 initrd.

BusyBox 官方没有提供编译好的 arm64 版本. 方便起见可以直接下载[助教编译好的 binary](tba). [源码下载](https://busybox.net/downloads/busybox-1.32.1.tar.bz2), 也可以自行编译.

### 制作 initrd

BusyBox 的特点之一是，如果你使用某个支持的别名来运行它，它就会像那条命令一样工作. 创建符号链接是最常见的以别名使用 BusyBox 的方式.

```
chmod 755 busybox
mkdir -p rootfs/bin/
mv busybox rootfs/bin/
cd rootfs/bin/
for item in $(./busybox --list)
  do
    ln -s busybox $item
  done
```

`rootfs/init`:

```shell
#!/bin/sh
/bin/sh
```

记得给这个 init 文件加上执行权限

```
chmod +x init
```

按照前面的教程将 rootfs/ 目录打包为 initrd.cpio.gz，并使用 QEMU 启动，你就有 shell 可以玩了 :D

hint: 有 `/bin/mknod`