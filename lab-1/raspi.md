# 在树莓派上运行 Linux 内核和 init (选做)

## 硬件设备额外要求

- HDMI 线和屏幕
- CP2102 USB 转串口模块

由于需求额外硬件，在树莓派实体机上运行为选做，欢迎感兴趣的同学积极尝试。如果选做此项请在实验报告中说明，批改时评测环境将变为树莓派实体机 (如果在实体机上和预期表现不符会再用QEMU评测). <del>虽然相比 QEMU 没有额外加分, 但是助教们会给你鼓掌掌.</del>

## 注意事项

建议阅读树莓派官方对于串口的[说明](https://www.raspberrypi.org/documentation/configuration/uart.md). 通过 GPIO 连接的是 mini UART, 对应设备 `/dev/ttyS0` .

## 流程

1. SD 卡上创建一个 fat32 分区, 并挂载
2. 将 `boot_utils` 里的内容, 复制到挂载的分区内
3. 将自己编译的 kernel 和 打包的 initrd 复制进挂载的分区内, 并命名为 `kernel8.img` 和 `initrd.cpio.gz`
4. `umount` 该分区, 再取出 SD卡
5. SD 卡放入树莓派, 连接显示器和串口模块, 打开串口终端 (如 Putty), 再接通电源.
6. 观察显示器和串口终端上的输出.

## 串口

### 串口配置

想要看到正确的串口输出, 需要:

1. `bootcode.bin` 中设置 `BOOT_UART=1` 
```bash
sed -i -e "s/BOOT_UART=0/BOOT_UART=1/" bootcode.bin
```

2. `config.txt` 中设置 `enable_uart=1` 

以上在提供的 `boot_utils` 中已经配置好.

### 串口连线

![uart](assets/../../assets/uart.jpg)

### 串口客户端

以 Putty 为例:

Speed 设为 115200, Serial line 设为 `/dev/ttyUSB0`, Connection type 为 Serial, 然后点击 open.

## 参考资料

- [bootmodes](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bootmodes/README.md)
- [uart](https://www.raspberrypi.org/documentation/configuration/uart.md)