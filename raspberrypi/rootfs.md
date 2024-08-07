# 完善rootfs

Linux系统就是一个内核加各种程序组成，这些常用的如ls, mkdir, ifconfi g命令都是一个个小程序，[构建树莓派镜像](Raspberry-Euler.md)一文中使用的busybox工具是一个工具集合，里面包含了一些最常用的小程序。
[源码分析](https://www.cnblogs.com/zhuangquan/p/11557781.html)
如果想选择默认功能更为丰富的rootfs构建工具，可以选择debian系的工具debootstrap，或者使用buildroot

- [完善rootfs](#完善rootfs)
  - [rootfs中的常用目录](#rootfs中的常用目录)
  - [/etc中的常用配置文件](#etc中的常用配置文件)
    - [inittab文件](#inittab文件)
    - [rcS文件](#rcs文件)
    - [fstab文件](#fstab文件)
    - [rc.local文件](#rclocal文件)
    - [profile文件](#profile文件)
  - [添加设备文件](#添加设备文件)
  - [配置dhcp自动获取ip地址](#配置dhcp自动获取ip地址)
    - [加入default.script配置文件](#加入defaultscript配置文件)
    - [开机自动启动dhcp服务](#开机自动启动dhcp服务)
    - [实现网线热插拔](#实现网线热插拔)
  - [添加用户管理](#添加用户管理)
    - [添加用户相关文件](#添加用户相关文件)
      - [账号文件](#账号文件)
      - [用户组文件](#用户组文件)
    - [设置开机自动登录](#设置开机自动登录)
    - [加强密码安全性](#加强密码安全性)
  - [配置远程](#配置远程)
    - [telnet](#telnet)
    - [ftp](#ftp)
    - [ssh](#ssh)
      - [编译依赖](#编译依赖)
      - [编译OpenSSH](#编译openssh)
    - [开机自动启动远程服务](#开机自动启动远程服务)
  - [添加包管理器](#添加包管理器)
  - [安装ffmpeg](#安装ffmpeg)
    - [包管理器安装](#包管理器安装)
    - [从源码构建](#从源码构建)
      - [下载源码](#下载源码)
      - [配置编译参数](#配置编译参数)
      - [安装](#安装)
  - [安装Valgrind](#安装valgrind)
    - [下载源码](#下载源码-1)
    - [修改配置](#修改配置)
    - [编译和安装](#编译和安装)
    - [测试valgrind](#测试valgrind)
  - [文件系统扩容](#文件系统扩容)
    - [磁盘分区扩容](#磁盘分区扩容)
    - [文件系统扩容](#文件系统扩容-1)

## rootfs中的常用目录

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

## /etc中的常用配置文件
Linux系统的配置文件都放置在/etc目录下，下面介绍几个常用的配置文件
### inittab文件
inittab文件是初始进程init的配置文件，内核引导完成后，内核会启动初始化进程init（用户级进程）来进行 系统的各项配置。init是系统第一个进程，它是系统上运行的所有其他进程的父进程，他会观察其子进程，并在需要时启动、停止、重启它们。init主要是 用来完成系统的各项配置。init从/etc/inittab获取所有信息。想了解BusyBox init及其inittab基本原理的，可以看[这篇文章](https://www.cnblogs.com/arnoldlu/p/10868354.html)
其中有几个action，定义方法如下表：

|方法|执行条件|说明|
|----|-------|----|
|sysinit|系统启动后执行|只执行一次，init进程等它结束之后执行其他动作|
|wait|系统执行完sysinit进程后|只执行一次，init进程等待它结束后执行其他动作|
|once|系统执行完wait进程后|只执行一次，init不等他结束|
|respawn|启动完once进程后|init进程发现其退出后，重新启动它|
|askfirst|启动完respawn后|与respawn类似，但init进程先输出Please Enter to activate this console，等用户按下回车后才启动子进程|
|shutdown|当系统关机时|重启，关闭系统命令时|
|restart|Busybox中配置CONFIG_FEATURE_USE_INITTAB，并且init进程收到SIGHUP信号时|先重新读取，解析inittab文件，再执行restart程序|
|ctrlaltdel|按下ctrl+alt+del时|-|

格式如下：

+ #开始的行是注释
+ 冒号在里面是分隔符，分隔开各个部分。
+ inittab内容是以行为单位的，行与行之间没有关联，每行都是一个独立的配置项，每一个 配置项表示一个具体的含义。
+ 每一行的配置项都是由3个冒号分隔开的4个配置值共同确定的。这四个配置值就是id:runlevels:action:process 值得注意得是有些配置值可以空缺，空缺后冒号不能空缺，所以有时候会看到连续2个冒号。
+ 每一行的配置项中4个配置值中最重要的是action和process，action是一个条件/状态，process是一个可被执行的程序的pathname。合起来的意思就是：当满足action的条件时就会执行process这个程序。

busybox官方给出的说明以及样例在/examples/inittab文件中
如果没有inittab，busybox会按照以下默认配置运行init
```sh
::sysinit:/etc/init.d/rcS # 执行rcS
::askfirst:/bin/sh #执行命令行
::ctrlaltdel:/sbin/reboot
::shutdown:/sbin/swapoff -a
::shutdown:/bin/umount -a -r
::restart:/sbin/init
tty2::askfirst:/bin/sh
tty3::askfirst:/bin/sh
tty4::askfirst:/bin/sh
```
inittab的关键就是：“当满足action的条件时就会执行process这个程序”。
### rcS文件
可以看到inittab的第一步即解析rcS文件，其在系统初始化阶段会被init进程调用，并且执行里面的命令，因此该文件用于执行一些系统初始化指令

```
mkdir rootfs/etc/init.d
sudo vim rootfs/etc/init.d/rcS
```

内容为：
```sh
#!/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
export PATH
umask 022

mount -a
mkdir /dev/pts
mount -t devpts devpts /dev/pts

hostname euler

ifconfig lo 127.0.0.1
```


### fstab文件
这个文件是供mount读取设备所用，在rcS中有mount -a命令，负责挂载系统所用到的设备，mount -a从fstab中读取参数
先加入三个内核自带的三个虚拟文件系统

```
#<file system>  <mount point>   <type>  <options>       <dump>  <pass>
proc            /proc           proc    defaults        0       0
tmpfs           /tmp            tmpfs   defaults        0       0
sysfs           /sys            sysfs   defaults        0       0
```
这三个目录为linux内核文件系统，存在于内存中
+ proc用于在系统运行时访问内核内部数据结构，改变内核设置，proc只存在于内存当中，以文件系统的方式为访问内核数据提供接口，其被挂在于根目录的/proc目录下。

+ sysfs将系统中的设备组织成层次结构，并向用户模式程序提供详细的内核数据结构信息

+ tmpfs用于存储临时文件，这些文件通常是由操作系统或应用程序创建的。这些文件可以是日志文件、临时缓存文件、程序临时文件、打印队列文件等

除这三个目录外，通常还需要再根据物理设备以及分区情况，加入其他文件系统。

### rc.local文件
/etc/rc.local是被init.d/rcS 文件调用执行的特殊文件，与Linux 系统硬件平台相关，如安装核心模块、进行网络配置、运行应用程序、启动图形界面等。
因此可以将rcS文件中非核心操作的内容移动到rc.local中，使得配置文件更加清晰

rcS文件修改如下：
```sh
#!/bin/sh

umask 022

PATH=/sbin:/bin:/usr/sbin:/usr/bin
export PATH

mount -a
mkdir /dev/pts
mount -t devpts devpts /dev/pts

exec /etc/rc.local
```
rc.local中的内容如下：
```sh
#!/bin/sh

hostname Euler

echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s

ifconfig lo 127.0.0.1
telnetd # 启动telnet服务供远程登录


echo "**************************************************"
echo "*                                                *"
echo "*                 Openeuler-22.04                *"
echo "*                                                *"
echo "*          Linux Kernel version 5.10.0           *"
echo "*                                                *"
echo "*                 2024-07-03                     *"
echo "*                                                *"
echo "**************************************************"
```

### profile文件
profile文件在每个用户登录时被执行一次，用于设置每个用户的环境信息，root用户的环境信息由init进程设置，而其他用户则通过login进程读取profile文件设置
```sh
PATH=/bin:/sbin:/usr/bin:/usr/sbin
LD_LIBRARY_PATH=/lib:/usr/lib

HOSTNAME=`/bin/hostname`
PS1='\u@\h:\w\$ '

export PATH LD_LIBRARY_PATH HOSTNAME PS1
```
## 添加设备文件
busybox中自动利用mdev管理设备，在编译时已经设置好，其工作过程参考这个[博客](http://www.360doc.com/content/10/0428/11/496343_25245348.shtml)


## 配置dhcp自动获取ip地址

udhcpc的[工作过程](https://www.cnblogs.com/arnoldlu/p/13567937.html)
### 加入default.script配置文件
将busybox的默认配置复制到根文件系统中，以正确使用udhcpc获取ip地址
具体解析可以看这个[博客](https://www.cnblogs.com/arnoldlu/p/13567937.html)
```
mkdir rootfs/usr/share/udhcpc
cp busybox-1.36.1/examples/udhcp/simple.script rootfs/usr/share/udhcpc/default.script
```
### 开机自动启动dhcp服务
在rcS文件的末尾添加
```sh
udhcpc
```
### 实现网线热插拔
http://blog.chinaunix.net/uid-16759545-id-4891833.html
https://blog.csdn.net/xuesong10210/article/details/113858728

## 添加用户管理
### 添加用户相关文件
linux中，用户管理涉及到两个文件：账号文件 /etc/passwd和用户组文件 /etc/group。

#### 账号文件
首先先创建账号文件
```
vim /etc/passwd
```
添加内容
```
root:x:0:0:root:/root:/bin/sh
```
账号文件的每一行是一个用户和其对应的设置，用":"隔开，每字段的含义为
> 用户名称：用户密码：USER ID：GROUP ID：相关注释：主目录：使用的shell

1. 用户名称：在Linux系统中用唯一的字符串区分不同的用户，用户名可以由字母、数字、下划线组成。注意Linux系统中对字母大小写是敏感的。 比如USERNAME1 和username1是两个不同的用户。
2. 用户密码：在用户校验时验证用户的合法性。超级用户root可以更改系统中所有用户的密码，普通用户登陆后可以使用 pawwd 命令来更改自己的密码。在 /etc/passwd 文件中该字段一般为 x ，或加密存储的密码，处于安全考虑，一些高级发行版中该字段加密后的密码数据已经移至 /etc/shadow 中。 /etc/shadow 文件不能被普通用户读取，只有超级用户才有权读取。而此处使用的busybox依然在此字段存放加密的密码。
3. 用户标识号(USER ID)：USER ID，简称UID，是一个数值，用于唯一标识Linux系统中的用户。在Linux系统中最多可使用65535个用户名，用户名和UID都可以用于标识用户。相同UID的用户可以认为是同一用户，同时他们也具有相同的权限，当然对于使用者来说用户名更容易记忆和使用。
4. 组标识号(GROUP ID)：GROUP ID，检查GID，这是当前用户所属的默认用户组标识。当添加用户时，系统会默认建立一个和用户名一样的用户组，多个用户可以属于相同的用户组。用户可以同时属于多个组，每个组也可以有多个用户。除了在 /etc/passwd 文件中指定其归属的基本组外，/etc/group 文件中也指明一个组所包含的用户。
5. 相关注释：用于存放一些用户的其它信息，比如用户含义说明、用户地址信息等。
6. 主目录：该字段定义了用户的主目录，登录后shell将把该目录作为用户的工作目录。登录系统后可以使用 pwd 命令查看。超级用户 root的工作目录为 /root。每个用户都有自己的主目录，默认一般在 /home 下建立与用户名一致的目录，同时建立目录时可以指定其它目录作为用户的主目录。
7. 使用的shell：shell是当前用户登录系统是运行的程序的名称，通常是 /bin/bash。同时系统中可能存在其它的shell，比如 tsh。用户可以自己指定shell，也可以随时更改，比较流行的就行 /bin/bash，在busybox中一般是/bin/sh

#### 用户组文件
创建文件
```
vim /etc/group
```
添加内容
```
root:x:0:root
```
该文件用于保存用户组的所有信息，通过它可以更好地对系统中的用户进行管理。对用户分组是一种有效的手段，用户组和用户之间属于多对多的关系，一个用户可以属于多个组，一个组也可以包含多个用户。用户登录时默认的组存放在 /etc/passwd 中。

用户组文件也是每行代表一个用户组的信息，含义为
>用户组名：用户组密码：用户组标识号：组内用户列表

1. 用户组名：可以由字母、数字、下划线组成，用户组名是唯一的，和用户名一样不可重复。
2. 用户组密码：该字段存放的是用户组加密后的密码，这一字段基本不使用，Linux系统的用户组都没有密码。
3. 用户组标识号：GROUP ID 简称GID，和用户标识号UID类似，也是一个整数，用于唯一标识一个用户组。
4. 组内用户列表：是属于这个组的所有用户的列表，不同用户之间用逗号分隔，不能有空格。这个用户组可能是用户的主组，也可能是附加组。

至此系统中已经默认自带root用户，并且可以在系统运行之后新建，切换用户。但在开机时依然不会登录，直接选择进入命令行

### 设置开机自动登录
开机自动登录功能需要修改inittab文件来实现，即在运行shell之前先运行一个login程序，将inittab中的askfirst:/bin/sh替换为，即先启动getty，然后再运行login进程来进行登录
```sh
console::respawn:/sbin/getty -L -n -l /etc/autologin console 0 vt100
```
然后添加autologin脚本
```sh
#!/bin/sh
/bin/login -f root
```
其中选项-f表示不对root用户进行验证
系统的登录流程如下：

### 加强密码安全性

## 配置远程
### telnet
telnet服务器由busybox中的telnetd程序来实现，默认端口为23端口
### ftp
ftp服务为ftpd


### ssh
OpenSSH（OpenBSD Secure Shell）是使用SSH透过计算机网络加密通信的实现。它是取代由SSH Communications Security所提供的商用版本的开放源代码方案。目前OpenSSH是OpenBSD的子项目。
OpenSSH常常被误认以为与OpenSSL有关系，但实际上这两个项目有不同的目的，不同的发展团队，名称相近只是因为两者有同样的软件发展目标──提供开放源代码的加密通信软件。
而目前最经常的应用：SSH远程连接。相比telnet远程连接，SSH更安全

下面将从源码构建OpenSSH

#### 编译依赖

[OpenSSL](https://openssl-library.org/source/)

[zlib](https://www.zlib.net/)

**编译Zlib**
```sh
export WORK_DIR `pwd`
export CHOST arm-linux-gnueabihf #配置交叉编译选项
./configure --prefix=${WORK_DIR}/../zlib_install
make -j10
make install
```
这一步很多教程让自己修改Makefile，但是修改Makefile很麻烦还容易出错，故不推荐使用这种方法。阅读zlib的configure文件发现其通过CHOST环境变量来控制交叉编译选项，因此只需将CHOST设置为交叉编译前缀即可。

下面是configure文件中控制交叉编译的部分：
```sh
# set command prefix for cross-compilation
if [ -n "${CHOST}" ]; then
    uname=${CHOST}
    mname=${CHOST}
    CROSS_PREFIX="${CHOST}-"
else
    mname=`(uname -a || echo unknown) 2>/dev/null`
fi
```

**编译OpenSSL**
```sh

./Configure --prefix=${WORK_DIR}/../openssl_install --cross-compile-prefix=arm-linux-gnueabihf- no-asm no-shared linux-armv4
make -j10
make install
```
#### 编译OpenSSH


### 开机自动启动远程服务
既然都使用远程了，我们肯定不希望先连接显示器键盘，输入命令开启远程服务后，再进行远程连接，因此远程服务需要在开机时自动启动。
在inittab中设置一个once，来启动telnet和ftp，其中网络的基本设置在sysinit中完成，而telnet和ftp服务器进程需要等网络基本设置完成后启动，并且持续运行，因此使用once活动来实现
```sh
::once:/etc/initremote
```
/etc/initremote内容如下：
```sh
#!bin/sh
telnetd
tcpsvd -vE 0.0.0.0 21 ftpd -w /home/euler &
```

## 添加包管理器

## 安装ffmpeg
### 包管理器安装
ffmpeg是一个开源的音视频编解码工具，官方文档在[这里](https://trac.ffmpeg.org/)如果有包管理工具可以直接通过包管理工具安装
```sh
sudo apt install ffmpeg
```
如果没有包管理器，也可以通过源码构建

### 从源码构建
#### 下载源码
新建一个工作目录ffmpeg，然后下载源码。
```sh
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
```
下载好后还有一个ffmpeg目录，进入这个目录
```sh
cd ffmpeg
```
#### 配置编译参数
   
ffmpeg的编译参数通过configure脚本设置，先执行一下，发现输入--help命令可以查看帮助，查看一下这些参数的含义，根据需求选择，在此处我们往32位的树莓派上交叉编译。
```sh
./configure --arch=arm\
 --cross-prefix=arm-linux-gnueabihf-\
 --target-os=linux\
 --prefix=/home/anakin/Data/code/euler/rootfs/usr\
 --disable-ffprobe\
 --enable-mmal\
 --enable-omx-rpi\
 --enable-gpl\
 --enable-nonfree
```
  首先先设置目标架构arm，交叉编译工具armhf，目标系统Linux。

  --prefix为euler系统的根文件系统的/usr目录，这一步非常重要，否则后面会安装到自己的电脑上。

  然后裁剪掉一些不需要的库，比如媒体分析工具ffprobe

  最后新加入一些需要的库：
+ --enable-mmal：Braodcom的多媒体硬件优化层，可以为树莓派开启硬件加速
+ --enable-omx-rpi：开启树莓派的OpenMAX加速
+ --enable-gpl:启用gpl许可的编码器
+ --enable-nonfree：启用非自由的编码器


如果提示
>WARNING: arm-linux-gnueabihf-pkg-config not found, library detection may fail.
则安装pkg-config即可
```sh
sudo apt install pkg-config-arm-linux-gnueabihf
```
都完成后开始编译
```sh
make -j10
```
#### 安装
编译完成之后，用make install命令安装即可
```sh
make install
```
## 安装Valgrind 
Valgrind 是一款用于内存调试，内存泄漏检测以及性能分析，检测线程错误的开发工具。

### 下载源码
在[官网](https://valgrind.org/downloads/)下载对应的包，然后解压，得到一个目录valgrind，进入这个目录
```
wget https://sourceware.org/pub/valgrind/valgrind-3.23.0.tar.bz2
tar -xvf valgrind-3.23.0.tar.bz2
cd valgrind-3.23.0
```
### 修改配置
查看目录里面的README和README.aarch64（没有arm32的，看着这俩猜一猜怎么设置交叉编译工具链），然后再打开configure脚本看看。
看到大概有这样一段和交叉编译相关的内容
```
Development so far was however done by cross compiling, viz:
  export CC=aarch64-linux-gnu-gcc
  export LD=aarch64-linux-gnu-ld
  export AR=aarch64-linux-gnu-ar

  ./autogen.sh
  ./configure --prefix=`pwd`/Inst --host=aarch64-unknown-linux \
              --enable-only64bit
  make -j4
  make -j4 install

Doing this assumes that the install path (`pwd`/Inst) is valid on
both host and target, which isn't normally the case.  To avoid
this limitation, do instead:

  ./configure --prefix=/install/path/on/target \
              --host=aarch64-unknown-linux \
              --enable-only64bit
  make -j4
  make -j4 install DESTDIR=/a/temp/dir/on/host
  # and then copy the contents of DESTDIR to the target.
```
大概意思就是，先设置交叉编译工具链，然后运行autogen，然后利用configure配置

我们首先需要修改交叉编译工具为arm32位的交叉编译工具：
```sh
export CC=arm-linux-gnueabihf-gcc
export LD=arm-linux-gnueabihf-ld
export AR=arm-linux-gnueabihf-ar
```
然后修改configure里面的host为arm-linux，并且去掉only64bit，因为我们是32bit。

注意这个--prefix参数，这里要填写的是在目标机器上的安装路径，在这里我们安装到/usr路径下，所以可以得到命令
```sh
./autogen.sh
./configure --prefix=/usr --host=arm-linux
```
至此就完成了编译前配置，设置完后会列出相应的配置：
```
 Version: 3.23.0
         Maximum build arch: arm
         Primary build arch: arm
       Secondary build arch:
                   Build OS: linux
     Link Time Optimisation: no
       Primary build target: ARM_LINUX
     Secondary build target:
           Platform variant: vanilla
      Primary -DVGPV string: -DVGPV_arm_linux_vanilla=1
         Default supp files: ./xfree-3.supp ./xfree-4.supp glibc-2.X-drd.supp glibc-2.X-helgrind.supp glibc-2.X.supp
```
确认无误后开始编译
### 编译和安装
```sh
make -j4
make -j4 install DESTDIR={$ROOTFS_DIR} #直接放到rootfs里面，因为自带前缀/usr，故不用带/usr路径
```
如果编译过程中出现问题：
>sorry, unimplemented: Thumb-1 hard-float VFP ABI

打开configure文件，把里面的-march=armv6修改成-marm即可

上下文大概如下：
```sh
case "${host_cpu}" in
...)
  ......
arm*)
        { $as_echo "$as_me:${as_lineno-$LINENO}: result: ok (${host_cpu})" >&5
$as_echo "ok (${host_cpu})" >&6; }
        VGCONF_PLATFORM_ARM_ARCH="-marm" #这里原来是-march=armv6
        ARCH_MAX="arm"
        ;;
```
将rootfs同步到树莓派，进行测试即可

### 测试valgrind
参考这几篇博客

http://blog.chinaunix.net/uid-23629988-id-3033741.html
https://blog.csdn.net/weixin_45518728/article/details/119865117


## 文件系统扩容
从镜像刷写到SD卡后，rootfs的容量和镜像中的划分大小是一样的，也就是只有500多M的空间可用，因此需要将root分区扩容至SD卡剩余的所有空间。因此需要将磁盘逻辑分区扩大，然后将文件系统也调整至相应容量

以下操作都需要离线进行，即将树莓派系统的SD卡插到电脑上进行。
### 磁盘分区扩容
用fdisk -l命令查看SD卡的物理空间和分区表。
```sh
fdisk -l
```
根据容量找到SD卡的盘符，可以看到我的是/dev/sdc，并且/dev/sdc3分区是root分区，但是只有502M的可用空间，显然没有占满整个SD卡
![](../img_folder/Raspberry/fdisk.png)

下面需要用fdisk工具，将root对应的分区扩容至所有剩余空间，整个操作思路为，删除原有分区，然后再新建一个和原root分区起始块号相同，大小为SD卡中全部可用块的分区。
```sh
sudo fdisk /dev/sdc #sdc换成你对应的设备号
```
进入菜单之后：
1. 按p，查看分区表，会看到一个和上面图片差不多的内容，比如我的是这样
```
Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1  *       8192  593919  585728  286M  c W95 FAT32 (LBA)
/dev/sdc2        593920 1593343  999424  488M 82 Linux swap / Solaris
/dev/sdc3       1593344 2621439 1028096  502M 83 Linux
```

记下Linux类型分区的Start块号，后面有用

2. 按d，删除分区3
   
3. 按n，新建分区，分区类型选择主分区p，分区号为3，起点为上面记录的起始块号，终点为默认（最后一块），最后移除签名选N
   
4. 输入w保存分区

现在就完成了磁盘逻辑分区的调整，可以再次使用fdisk -l命令查看分区表
### 文件系统扩容
磁盘分区扩容完成后，需要修改文件系统数据结构，使其能识别全部的分区块

先用e2fsck命令检查分区表

```sh
sudo e2fsck /dev/sdc3
```
e2fsck工具常用选项如下：
```
-a：不询问使用者意见，便自动修复文件系统；
-b<superblock>：指定superblock，而不使用预设的superblock；
-B<区块大小>：指定区块的大小，单位为字节；
-c：一并执行badblocks，以标示损坏的区块；
-C：将检查过程的信息完整记录在file descriptor中，使得整个检查过程都能完整监控；
-d：显示排错信息；
-f：即使文件系统没有错误迹象，仍强制地检查正确性；
-F：执行前先清除设备的缓冲区；
-l<文件>：将文件中指定的区块加到损坏区块列表；
-L<文件>：先清除损坏区块列表，再将文件中指定的区块加到损坏区块列表。因此损坏区块列表的区块跟文件中指定的区块是一样的；
-n：以只读模式开启文件系统，并采取非互动方式执行，所有的问题对话均设置以"no"回答；
-p：不询问使用者意见，便自动修复文件系统；
-r：此参数只为了兼容性而存在，并无实际作用；
-s：如果文件系统的字节顺序不适当，就交换字节顺序，否则不做任何动作；
-S：不管文件系统的字节顺序，一律交换字节顺序；
-t：显示时间信息；
-v：执行时显示详细的信息；
-V：显示版本信息；
-y：采取非互动方式执行，所有的问题均设置以"yes"回答。
```

用resize2fs工具修改ext4文件系统索引
```sh
sudo resize2fs /dev/sdc3
```

至此便完成了扩容，可以将/dev/sdc3挂载后，使用df -h命令查看分区的大小验证操作是否成功。