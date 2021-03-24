# Lab 0

为了让同学们能够更好地学习 2021 春季学期的 OSH 课程，我们准备了一些预备内容作为 Lab 0，同学们可以自行学习。

Lab 0 这并非正式实验，不会计入实验分，但请大家了解、学习以下内容。

## Linux 环境

如果你的主力系统是 Linux，<del>欢迎你加入 LUG</del>你可以跳过这个部分。

如果你内存充足、或者不太想在物理机上安装 Linux，可以尝试使用虚拟机运行 Linux。具体可以参照[在 Windows 中使用虚拟机（By iBug）](https://ibug.io/cn/2019/02/setup-ubuntu-in-vmware/)、[在 macOS 中使用虚拟机（By Taoky）](https://blog.taoky.moe/2019-02-23/installing-os-on-vm.html)这两篇文章。

如果你想尝试把本地系统换成 Linux、或者双系统，可以参考[各个 Linux 发行版的介绍](https://101.lug.ustc.edu.cn/Ch01/#linux-distributions)，选择合适的发行版进行安装。这里推荐根据 [Arch 安装教程](<https://wiki.archlinux.org/index.php/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)安装一次 Arch Linux，这样你就可以详细地了解 Linux 各部分的组成。

Linux 最基本的使用也可以从 [LUG Linux 101](https://101.lug.ustc.edu.cn/) 开始学习。

如果想了解更多有关 Linux 的技巧，可以看 [Debian 教程](https://www.debian.org/doc/manuals/debian-reference/ch01.zh-cn.html)或者 [Arch Linux 文章索引](<https://wiki.archlinux.org/index.php/General_recommendations_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)。

预期结果：有一个可用的 Linux 工作环境（不限于本地），和基本的 Linux 使用能力。

## 体验 Rust

参考 [Rust 安装页面](https://www.rust-lang.org/tools/install)，使用 Rustup 在本地快速安装 Rust 工具链。

然后体验 Cargo TODO

## 使用 Git

还没接触过 Git 的同学可以看看 [Git 入门教程](https://vlab.ustc.edu.cn/docs/tutorial/git/)，它包含基础的操作以及连接 GitHub/GitLab 远程仓库，和 Github 的 [Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet/)。

如果你对 Git 有一定了解，但是遇到麻烦、不知如何操作的时候可以看[这个](https://ohshitgit.com/zh)，它写了在一些令人不知所措的情景下应当执行的 Git 命令，这些情况比较难直接在 Git 的官方 docs 里查到。

Git 官方文档 [Pro Git book](https://git-scm.com/book/en/v2) 不太推荐作为入门教程，但是感兴趣可以去看看。

预期结果：能用 Git 对自己写的代码进行简单版本控制，使用 Commit 正确的文件并留下恰当的 Commit message，能在 GitHub 上进行多人合作。

## 玩一玩 Raspberry Pi

树莓派拿了干嘛？能吃？<del>能吃灰？</del>

如果你的树霉派正在躺着吃灰，可以参考[一名助教的博客](https://yyw.moe/2021/01/18/Raspberry-pi-init/)，烧个系统进去并探索一番。

预期结果：了解树莓派系统烧录、使用的过程。Have fun :D

## 认识 Makefile

这里有一篇 [OSH2020 助教 iBug 的博客](https://ibug.io/blog/2019/02/bootstrapping-make/) 初步介绍了 `Makefile`，你还可以阅读搜索 "Makefile cheatsheet" 的结果以快速了解更多，比如[这个 Cheat Sheet](https://bytes.usc.edu/cs104/wiki/makefile)。

预期结果：能理解别人写的 Makefile，能改写 Makefile。
