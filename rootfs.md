# 完善rootfs

Linux系统就是一个内核加各种程序组成，这些常用的如ls, mkdir, ifconfig命令都是一个个小程序，[构建树莓派镜像](Raspberry-Euler.md)一文中使用的busybox工具是一个工具集合，里面包含了一些最常用的小程序。

如果想选择默认功能更为丰富的rootfs构建工具，可以选择debian系的工具debootstrap，或者使用buildroot

- [完善rootfs](#完善rootfs)
  - [设置rcS文件](#设置rcs文件)
  - [创建/etc/fstab文件](#创建etcfstab文件)
  - [添加设备文件](#添加设备文件)
  - [配置dhcp自动获取ip地址](#配置dhcp自动获取ip地址)
  - [添加openssh和sftp](#添加openssh和sftp)
  - [添加用户管理](#添加用户管理)
  - [添加包管理器](#添加包管理器)


|目录 |含义 | 用途 |
|-----|-----|-----|
|/bin|重要的二进制 (binary) 应用程序|	包含二进制文件，系统的所有用户使用的命令都在这个目录下。|
/boot|	启动 (boot) 配置文件|	包含引导加载程序相关的文件
/dev|	设备 (device) 文件|	包含设备文件，包括终端设备，USB或连接到系统的任何设备
/etc|	配置文件、启动脚本等(etc)|	包含所有程序所需的配置文件，也包含了用于启动/停止单个程序的启动和关闭shell脚本
/home|	本地用户主 (home) 目录|	所有用户用home目录来存储他们的个人档案
/lib|	系统库 (libraries) 文件|	包含支持位于/bin和/sbin下的二进制文件的库文件
/lost+found|	在根 (/) 目录下提供一个遗失+查找(lost+found) 系统|	必须在root用户下才可以查看当前目录下的内容
/media|	挂载可移动介质 (media)|，诸如 CD、数码相机等	用于挂载可移动设备的临时目录
/mnt|	挂载 (mounted) 文件系统|	临时安装目录，系统管理员可以挂载文件系统
/opt|	提供一个供可选的 (optional) 应用程序安装目录|	包含从各个厂商的附加应用程序，附加的应用程序应该安装在/opt或者/opt的子目录下
/proc|	特殊的动态目录，用以维护系统信息和状态，包括当前运行中进程 (processes) 信息|	包含系统进程的相关信息，是一个虚拟的文件系统，包含有关正在运行的进程的信息，系统资源以文本信息形式存在
/root|	root (root) 用户主文件夹| 读作“slash-root”	
/sbin|	重要的系统二进制 (system binaries) 文件|	也是包含的二进制可执行文件。在这个目录下的linux命令通常都是由系统管理员使用的，对系统进行维护
/sys|	系统 (system) 文件|	
/tmp|	临时(temporary)文件|	包含系统和用户创建的临时文件。当系统重启时，这个目录下的文件将都被删除
/usr|	包含绝大部分所有用户(users)都能访问的应用程序和文件|	包含二进制文件，库文件。文档和二级程序的源代码
/var|	经常变化的(variable)文件，诸如日志或数据库等|	代表变量文件。在这个目录下可以找到内容可能增长的文件

## 设置rcS文件
/etc/init.d/rcS文件在系统初始化阶段会被init进程调用，并且执行里面的命令，因此该文件用于执行一些系统初始化指令

```
mkdir rootfs/etc/init.d
sudo vim rootfs/etc/init.d/rcS
```

内容为：
```
#!/bin/sh

umask 022

mount -n -t proc proc /proc
mount -n -t sysfs sysfs /sys 
mount -n -t tmpfs tmpfs /tmp

hostname euler

ifconfig lo 127.0.0.1
```
挂载的三个目录为linux内核文件系统，存在于内存中
其中proc，用于在系统运行时访问内核内部数据结构，改变内核设置，proc只存在于内存当中，以文件系统的方式为访问内核数据提供接口，其被挂在于根目录的/proc目录下。

sysfs将系统中的设备组织成层次结构，并向用户模式程序提供详细的内核数据结构信息

tmpfs用于存储临时文件，这些文件通常是由操作系统或应用程序创建的。这些文件可以是日志文件、临时缓存文件、程序临时文件、打印队列文件等

## 创建/etc/fstab文件
这个文件是供mount读取设备所用。
```
 
#<file system>  <mount point>   <type>  <options>       <dump>  <pass>
proc            /proc           proc    defaults        0       0
tmpfs           /tmp            tmpfs   defaults        0       0
sysfs           /sys            sysfs   defaults        0       0

```
## 添加设备文件
busybox中自动利用mdev管理设备，在编译时已经设置好，其工作过程参考这个[博客](http://www.360doc.com/content/10/0428/11/496343_25245348.shtml)

## 配置dhcp自动获取ip地址

## 添加openssh和sftp

## 添加用户管理

## 添加包管理器