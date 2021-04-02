# Lab 1

## 实验内容

- 了解树莓派的启动原理和过程
- 编译适用于树莓派的内核
- 构建初始内存盘
- 裁剪内核并在 QEMU 或树莓派中测试
- 了解 BIOS Boot 的基本流程和实现方案

## 实验要求

提醒: 编译一次内核的时间可能较久, 建议合理安排时间, 不要当 ddl 人.

### 准备工作

- Linux 环境和基本使用
- 使用 git 管理自己的实验目录
- 创建 GitHub 私有仓库并提交
- 了解树莓派启动的基本过程

推荐先阅读 lab0 内容 (Rust 相关可以暂缓).

### 目录结构

```
.                         // Git 根目录
├── lab1                  // 实验一根目录
│   ├── docs              // 实验一实验报告根目录
│   │   └── *.md          // 文件名自拟，可以写在一个或多个 Markdown 文件中
│   ├── linux
│   │   ├── Image         // 内核文件
│   │   ├── initrd.cpio.gz// 初始内存盘压缩文件
│   │   ├── .config       // 内核编译配置文件
|   |   └── src
|   |       └── init.c    // 如果 init 是 binary, 在此处提交源码
│   └── boot
│       └── bootloader.img// 可启动的 bootloader 镜像
├── .gitignore            // 使用此文件忽略你不想提交的文件
└── README.md             // 写入姓名学号
```

### 评分标准

本次实验共 10 分:

- Git & GitHub 仓库:
  - 正确创建了 `OSH-2021-TA` 这个 GitHub 用户能访问的 private 仓库 `OSH-2021-Labs`.
  - 正确使用 git. 没有通过 GitHub 网页版大量上传文件; 合理进行版本管理, 不要在一两个 commit 中提交所有文件; commit message 具有一定可读性; 符合目录结构要求. (0.5 分)
- Linux 内核:
  - 正确编译出适用于树莓派的内核 (1 分)
  - 内核裁剪过, 并且大小不超过 10 MiB, 并且它可以完成以下「初始内存盘」中的全部任务, 如果你没有完成「初始内存盘」中的全部任务，助教会使用自己的  `initrd.cpio.gz` 文件进行测试) (1 分)
  - 在上一条的基础上，裁剪得到的内核大小不超过 6 MiB (0.5 分)
- 初始内存盘:
  - 正确构建了 `initrd.cpio.gz` 文件. (0.5 分)
  - 使用你编译的内核和 `initrd.cpio.gz`，其中的 `/init` 能够成功依次执行助教提供的程序 1 (0.5 分)
  - 使用你编译的内核和 `initrd.cpio.gz`，其中的 `/init` 能够成功依次执行助教提供的程序 1、2、3, 程序 3 执行完成后不出现内核恐慌 (Kernel Panic) (1 分)
- 初识 Boot:
  - 正确构建了能够启动的 `bootloader.img` 文件 (1 分)
  - 从第一组问题中选择两个进行回答 (1 分)
  - 从第二组问题中选择一个进行回答 (1 分)
- 思考题:
  - 在下面给出的思考题中任选 4 道并完成它们 (2 分)

## 思考题:

1. 请简要解释 `Linux` 与 `Ubuntu`, `Debian`, `ArchLinux`, `Fedora` 等之间的关系和区别.
2. [树莓派官方编译内核的文档](https://www.raspberrypi.org/documentation/linux/kernel/building.md) 中提到 `install the kernel modules onto the SD card`, 完成本实验是否需要这么做? 为什么? 对内核裁剪有什么指导意义? (hint: `menuconfig` 中标 `[M]` 的选项; `modules` 在启动过程中的作用, 和 `init` 的关系)
3. 简述树莓派启动的各个阶段 (hint: `bootcode.bin`, `start.elf`, `cmdline.txt` 等的作用)
4. 在树莓派启动阶段使用串口观察输出需要哪些条件? 为什么实验用树莓派上通过 GPIO 使用的串口是 miniUART?   
5. 为什么在 `x86` 环境下装了 `qemu-user` 之后可以直接运行 `arm64` 的 `busybox`, 其中的具体执行过程是怎样的? `qemu-user` 和 `qemu-system` 的主要区别是什么?
6. 查阅 `PXE` 的资料，从技术上描述它的功能是什么 (用自己的话描述)
7. 查阅 `GRUB` 的资料，从技术上描述它的功能是什么 (用自己的话描述)
8. 说明 `UEFI Boot` 的流程，并用截图指出你的某一个系统的 `EFI` 分区中包含哪些文件.
