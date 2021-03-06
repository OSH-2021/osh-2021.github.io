## 初识 Boot

对于一台（比较现代的）计算机，从上电到允许用户操作确实（在大多数时候）是一个漫长的过程。
然而如果我们愿意选用一些简单的方案，这个过程将不那么复杂。
这里我们就将以 x86 架构上的 BIOS Boot 为例，探索 Boot 的过程。

首先请自行查阅 x86 汇编的资料，了解一些基本的汇编知识，包括：

- 寄存器和栈是什么？如何操作？
- 如何使用 BIOS 中断调用（BIOS interrupt call）？如何基本地调用函数？如何基本地使用宏？
- 如何声明一个像常量的东西？如何声明一个像变量的东西？

不要纠结在上述问题上，大致明白即可。

接下来我们将查看一份不完全、但能工作的 bootloader 的代码：

首先是环境配置。
请确保你目前使用的是**具有图形界面的 Ubuntu 虚拟机**（如果使用其他发行版，请参照下方自行处理，一般不会出现奇怪的问题），运行以下指令：

```bash
# 安装工具
sudo apt install nasm qemu make
```

然后从此处获得代码：[bootloader-x86.tar.xz](https://ftp.lug.ustc.edu.cn/misc/osh/bootloader-x86.tar.xz) 并解压获得源码。

这份代码的入口是 `boot.asm` 文件，之后会运行 `loader.asm` 文件。`kernel.asm` 不会被运行。
其余两个 `fat12.inc` 和 `log.inc` 文件可以类比做 C 中的头文件、会被宏内连到前面的两个文件中。
此代码带有一份 `Makefile`，记载了如何完成编译和测试。
请阅读 `Makefile` 内容，使用正确的指令完成编译。（1 分）

现在，请从以下几个问题中选择 2 个（共 1 分），查阅相关资料后简要回答：

- `xor ax, ax` 使用了异或操作 `xor`，这是在干什么？这么做有什么好处呢？
- `jmp $` 又是在干什么？
- `[es:di]` 是一个寻址操作，这是在寻址内存的哪个地址呢？
- `boot.asm` 文件前侧的 `org 0x7c00` 有什么用？
- `times 510 - ($ - $$) db 0` 是在干什么？为什么要这么做呢？为什么这里有一个 `510`？

再在从以下几个问题中选择 1 个（1 分），简单了解代码流程后简要回答：

- 尝试修改代码，在目前已有的输出中增加一行输出“I am OK!”，样式不限，位置不限，但不能覆盖其他的输出；
- 通过 `boot.asm` 文件及相应的其他资料，简单描述 BIOS Boot 是怎么确定一个软盘（floppy）是可启动（bootable）的？
- 为什么 `boot.asm` 执行之后可以继续执行 `loader.asm`？
- 为什么要用 `boot.asm` 和 `loader.asm` 两个文件？把代码写在一个文件里不好吗？

再接下来几个问题不会给分，但如果你做出来了助教会为你鼓掌：

- 这些代码是不完全的，尝试补全这些代码，使得 `kernel.asm` 可以被运行；
- 同样是尝试补全这些代码，使得系统进入长模式（long mode）；
- 又同样是尝试补全这些代码，使得系统的运行可以跳转到你自行增添的 C/C++/Rust 代码上；
- 或者自行实现另一套的 UEFI Boot；
