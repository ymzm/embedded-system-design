# <center>嵌入式系统设计</center>  
# <center>实验报告 </center>  

## 1.实验名称  
Linux系统移植  
## 2.实验学时  
4学时  
## 3.实验内容和目的  
### 3.1 实验内容  
1、对开发板的内核进行配置、编译，并且生成设备树文件；    
2、在内核中添加网卡驱动，并对其做一些基本配置；    
3、在内核中添加SD卡驱动，并对其做一些基本配置；    

### 3.2 实验目的  
1、熟悉开发板内核的配置、编译过程，能够生成设备树文件；    
2、通过实验，了解如何在内核中添加网卡驱动及网络功能的基本配置；    
3、通过实验，了解如何在内核中添加SD卡驱动以及如何对其进行基本配置。    
## 4.实验原理  
Linux 内核主要功能包括：进程管理、内存管理、文件管理、设备管理、网络管理等。    
### 4.1 进程管理  
进程是在计算机系统中资源分配的最小单元。内核负责创建和销毁进程, 而且由调度程序采取合适的调度策略，实现进程之间的合理且实时的处理器资源的共享。从而内核的进程管理活动实现了多个进程在一个或多个处理器之上的抽象。内核还负责实现不同进程之间、进程和其他部件之间的通信。

### 4.2 内存管理  

内存是计算机系统中最主要的资源。内核使得多个进程安全而合理地共享内存资源，为每个进程在有限的物理资源上建立一个虚拟地址空间。内存管理部分代码可以分为硬件无关部分和硬件有关部分：硬件无关部分实现进程和内存之间的地址映射等功能；硬件有关部分实现不同体系结构上的内存管理相关功能并为内存管理提供硬件无关的虚拟接口。   


### 4.3 文件管理  

在 Linux 系统中的任何一个概念几乎都可以看作一个文件。内核在非结构化的硬件之上建立了一个结构化的虚拟文件系统，隐藏了各种硬件的具体细节。从而在整个系统的几乎所有机制中使用文件的抽象。Linux 在不同物理介质或虚拟结构上支持数十种文件系统。例如, Linux支持磁盘的标准文件系统 ext3 和虚拟的特殊文件系统。    

###  4.4 设备管理  

Linux 系统中几乎每个系统操作最终都映射到一个或多个物理设备上。 除了处理器, 内存等少数的硬件资源之外, 任何一种设备控制操作都由设备特定的驱动代码来进行。内核中必须提供系统中可能要操作的每一种外设的驱动。    

### 4.5 网络管理  

内核支持各种网络标准协议和网络设备。网络管理部分可分为网络协议栈和网络设备驱动程序。网络协议栈负责实现每种可能的网络传输协议（TCP/IP 协议等）；网络设备驱动程序负责与各种网络硬件设备或虚拟设备进行通讯。  Linux 内核源代码非常庞大，随着版本的发展不断增加。它使用目录树结构，并且使用 Makefile 组织配置编译。     

## 5.实验步骤  
### 5.1 内核的配置和编译  
#### 5.1.1 解压内核  
$ cd ~    
$ mkdir kernel    
$ cd kernel    
将“华清远见-嵌入式 ARM 实验箱资料-I：\实验代码\2、Linux 移植驱动及应用\2、Linux 系统移植\实验代码\内核移植”目录下的“linux-3.14.tar.bz2”拷贝到该目录下    
$ tar xvf linux-3.14.tar.xz    
$ cd linux-3.14    

#### 5.1.2 修改内核顶层目录下的 Makefile  
$ vi Makefile    
修改：    
ARCH  ?= $(SUBARCH)      
CROSS_COMPILE  ?= $(CONFIG_CROSS_COMPILE:"%"=%)    
为：    
ARCH ?= arm    
CROSS_COMPILE  ?= arm-none-linux-gnueabi-    

#### 5.1.3 拷贝标准板配置文件  
$ cp arch/arm/configs/exynos_defconfig .config    

#### 5.1.4 配置内核  
$ make menuconfig    
System Type --->    
(2) S3C UART to use for low-level messages    
该命令执行时会弹出一个菜单，我们可以对内核进行详细的配置。这里我们先查看一下，内核都提供了那些功能！  

#### 5.1.5 编译内核  
$ make uImage    
通过上述操作我们能够在 arch/arm/boot目录下生成一个 uImage 文件，这就是经过压缩的内核镜像。    
如果编译过程中提示缺少 mkimage 工具，需将上一章 uboot源码中的 tools/mkimage 拷贝到 ubuntu 的/usr/bin 目录下    
$ cp u-boot-2013.01/tools/mkimage /usr/bin    

#### 5.1.6 生成设备树  
生成设备树文件，以参考板 origen 的设备数文件为参考。 

$ cp arch/arm/boot/dts/exynos4412-origen.dts arch/arm/boot/dts/exynos4412-fs4412.dts    

添加新文件需修改 Makefile 才能编译    

$ vim arch/arm/boot/dts/Makefile   

在    
exynos4412-origen.dtb \    
下添加如下内容    
exynos4412-fs4412.dtb \    
编译设备树文件    
$ make dtbs    
拷贝内核和设备树文件到/tftpboot 目录下    
$ cp arm/arm/boot/uImage /tftpboot    
$ cp arch/arm/boot/dts/exynos4412-fs4412.dtb /tftpboot/    
在文件末尾添加下一行，地址为交叉工具链的绝对存放路径    
修改 uboot 启动参数    
重启板子在系统倒计时是按任意键结束启动，输入如下内容修改 uboot 环境变量：    
\# setenv serverip 192.168.9.120    
\# setenv ipaddr 192.168.9.233    
\# setenv bootcmd tftp 41000000 uImage\;tftp 42000000 exynos4412-fs4412.dtb\;bootm 41000000 –42000000    
\#setenv bootargs root=/dev/nfs nfsroot=192.168.9.120:/source/rootfs rw console=ttySAC2,115200    
init=/linuxrc ip=192.168.9.233    
\# saveenv    
注意：192.168.9.120 对应 Ubuntu 的 ip    
192.168.9.233 对应板子的 ip    
这两个 ip 应该根据自己的实际情况适当修改    
重启开发板查看现象      

### 5.2 以太网卡驱动移植  
运行 Ubuntu 12.04 系统，打开命令行终端，使用实验 14.2 的内核    
$ cd ~/kernel/ linux-3.14    
设备树文件修改    
$ vim arch/arm/boot/dts/exynos4412-fs4412.dts    
添加如下内容    
srom-cs1@5000000 {    
compatible = "simple-bus";    
\#address-cells = <1>;    
\#size-cells = <1>;    
reg = <0x5000000 0x1000000>;    
ranges;    
ethernet@5000000 {    
compatible = "davicom,dm9000";    
reg = <0x5000000 0x2 0x5000004 0x2>;    
interrupt-parent = <&gpx0>;    
interrupts = <6 4>;    
davicom,no-eeprom;    
mac-address = [00 0a 2d a6 55 a2];    
};    
};    
修改文件 driver/clk/clk.c    
修改    
static bool clk_ignore_unused;    
为    
static bool clk_ignore_unused = true;    
配置内核：    
make menuconfig    
[*] Networking support --->    
Networking options --->    
<*> Packet socket    
<*> Unix domain sockets    
[*] TCP/IP networking    
[*] IP: kernel level autoconfiguration    
Device Drivers --->    
[*] Network device support --->    
[*] Ethernet driver support (NEW) --->    
<*> DM9000 support    
File systems --->    
[*] Network File Systems (NEW) --->    
<*> NFS client support    
[*] NFS client support for NFS version 2    
[*] NFS client support for NFS version 3    
[*] NFS client support for the NFSv3 ACL protocol extension    
[*] Root file system on NFS    
编译内核和设备树   
$ make uImage    
$ make dtbs    
测试：    
拷贝内核和设备树文件到/tftpboot 目录下    
$ cp arm/arm/boot/uImage /tftpboot    
$ cp arch/arm/boot/dts/exynos4412-fs4412.dtb /tftpboot/    
启动开发板，修改内核启动参数，通过 NFS 方式挂载根文件系统，文件系统暂用“华清远见-嵌入式    
ARM 实验箱资料-I：\实验代码\2、Linux 移植驱动及应用\2、Linux 系统移植\实验代码\移植后的源码”下的 rootfs.tar.xz。   

### 5.3 SD卡驱动移植  
运行 Ubuntu 12.04 系统，打开命令行终端。  
$ cd ~/kernel/ linux-3.14  
修改设备树文件  
$ vim arch/arm/boot/dts/exynos4412-fs4412.dts  
修改  
sdhci@12530000 {  
bus-width = <4>;  
pinctrl-0 = <&sd2_clk &sd2_cmd &sd2_bus4 &sd2_cd>;  
pinctrl-names = "default";  
vmmc-supply = <&mmc_reg>;  
status = "okay";  
};  
为:  
sdhci@12530000 {  
bus-width = <4>;  
pinctrl-0 = <&sd2_clk &sd2_cmd &sd2_bus4>;  
cd-gpios = <&gpx0 7 0>;  
cd-inverted = <0>;  
pinctrl-names = "default";  
/*vmmc-supply = <&mmc_reg>;*/  
status = "okay";  
};  
配置内核  
$ make menuconfig  
Device Drivers --->  
<*> MMC/SD/SDIO card support --->  
<*> Secure Digital Host Controller Interface support  
<*> SDHCI support on Samsung S3C SoC  
File systems --->  
DOS/FAT/NT Filesystems --->  
<*> MSDOS fs support  
<*> VFAT (Windows-95) fs support  
(437) Default codepage for FAT  
(iso8859-1) Default iocharset for FAT  
-*- Native language support --->  
<*> Codepage 437 (United States, Canada)  
<*> Simplified Chinese charset (CP936, GB2312)  
<*> ASCII (United States)  
<*> NLS ISO 8859-1 (Latin 1; Western European Languages)  
<*> NLS UTF-8  
编译内核和设备树  
$ make uImage  
$ make dtbs  
$ cp arm/arm/boot/uImage /tftpboot  
$ cp arch/arm/boot/dts/exynos4412-fs4412.dtb /tftpboot/  
实验现象  

启动开发板，通过 NFS 方式挂载根文件系统。  
把 SD 卡插到 FS4412 的 SD 卡槽里，在串口终端可以看到如下图所示：  
[ 1.620000] mmc0: new high speed SDHC card at address cd6d  
[ 1.625000] mmcblk1: mmc0:cd6d SE08G 7.28 GiB  
[ 1.630000] mmcblk1: p1(mmcblk1 为设备名 p1 为分区名)  
挂载， 注意不要挂在 C eMMC  的分区  
\# mount -t vfat /dev/mmcblk1p1 /mnt  
查看/mnt/目录即可看到 sd 卡中内  

## 6.实验结果及分析  

## 7.实验结论、问题及改进建议  
本次实验，按照实验步骤进行，实验结果符合预期，实验过程比较顺利，但是有如下问题实验时不明白。

### 7.1 内核配置编译过程各个操作和生成文件的作用

实验比较顺利，但是我们一开始并不清楚内核配置编译过程各个操作和生成文件的作用。查阅资料并学习研究后，明白了以下文件的作用

#### 7.1.1  .config文件与uImage 

实验中我们使用make menuconfig 使用图形界面方式生成了.config文件。.config文件是构建自身内核所需的内核配置目录，它在CONFIG_XXX变量值中用y n m 三个状态决定是否构建内核。状态为y时，相应的二进制文件与vmlinux链接；状态为m时，不会和vmlinux链接，但是作为模块执行编译。内核配置是一项容易发生错误的庞大的工作，所以实验中我们找到了与开发板SoC相符合的自定义配置文件，大大提高了准确性并减少了工作量。

使用上述生成的配置文件，经过编译、链接、清除多余符号、压缩，再加上引导程序加载项（内核初始化head.o 解压缩misc.o）便生成了压缩内核映像文件zImage.

uImage则是使用工具mkimage对zImage加工而得。它是uboot专用的映像文件，在zImage之前加上一个长度为64字节的“头”，说明这个内核的版本、加载位置、生成时间、大小等信息；其0x40之后与zImage没区别。

#### 7.1.2 设备树文件

实验中我们对dts文件进行了修改，并使用make dtbs命令生成了dtb文件。设备树机制来负责传递硬件拓扑和硬件资源信息。主要向kernel传递platform的信息、runtime的配置参数、设备的拓扑结构以及特性。

#### 7.1.3 文件系统

根文件系统包括Linux启动时所必须的目录和关键性的文件，例如Linux启动时都需要有init目录下的相关文件，在 Linux挂载分区时Linux一定会找/etc/fstab这个挂载文件等，根文件系统中还包括了许多的应用程序bin目录等，任何包括这些Linux 系统启动所必须的文件都可以成为根文件系统。

实验中uImage文件和dtb文件制作完成之后，以nfs挂载方式就可以启动开发板了，但是启动后发现系统会卡住，是因为系统没有配置网卡驱动，无法通过nfs方式挂载根文件系统出错而退出启动。

### 7.2 驱动移植过程各个操作的作用及驱动设备和系统的关系

实验比较顺利，但是我们并不是很清楚驱动移植过程各个操作的作用及驱动设备和系统的关系。查阅资料并学习研究后，明白了以下内容

#### 7.2.1  驱动如何配置

make menuconfig 时候根据需要进行选择。

#### 7.2.2 物理设备如何配置

无法动态查找的物理设备需要根据实际情况和驱动的要求对dts文件进行修改，例如我们实验中的以太网卡。可以被动态查找的物理设备无需在dts中进行配置，例如鼠标、键盘、PCI网卡等。linux设备子系统以probe机制为每一个设备匹配驱动，匹配成功后驱动对设备进行初始化使设备可用。

## 小组成员  
SA20225161 付一豪  
SA20225151 段震伟  
SA20225140 戴航  

