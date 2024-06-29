# 在树莓派上安装Openeuler
# 安装64位OpenEuler
[安装方法可以直接参考文档](https://docs.openeuler.org/zh/docs/22.03_LTS_SP3/docs/Installation/%E5%AE%89%E8%A3%85%E5%9C%A8%E6%A0%91%E8%8E%93%E6%B4%BE.html)

[镜像源在gitee](https://gitee.com/openeuler/raspberrypi/tree/master)

# 安装32位OpenEuler
Euler官网不提供32位版本的树莓派镜像，因此需要自己参考文档构建
## 构造镜像

### 交叉编译内核
#### 安装依赖
```
sudo apt install bc bison flex libssl-dev make libc6-dev libncurses5-dev
```
32位：
```
sudo apt install crossbuild-essential-armhf
```
64位：
```
sudo apt install crossbuild-essential-arm64
```
#### 下载内核源码
先新建一个工作目录euler
```
mkdir euler
cd euler
```
此处选择openEuler 22.03 LTS SP1的树莓派kernel（就是个raspbian换壳）
```
git clone https://gitee.com/openeuler/raspberrypi-kernel.git -b openEuler-22.03-LTS-SP1 && cd raspberrypi-kernel
```
或选择原版的euler kernel
如果选择这个，在设置配置文件的时候记得手动设置一下
```
git clone https://gitee.com/openeuler/kernel.git -b OLK-6.6 && cd kernel
```
如果需要其他版本，请参考[官网](https://gitee.com/openeuler/raspberrypi/blob/master/documents/openEuler%E9%95%9C%E5%83%8F%E7%9A%84%E6%9E%84%E5%BB%BA.md#%E4%B8%8B%E8%BD%BD%E6%BA%90%E7%A0%81)自行选择

#### 设置环境变量
这一步主要是根据需要设置架构和交叉编译工具链到环境变量中

32位版本：
```
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
```
64位版本：
```
export ARCH=aarch64
export CROSS_COMPILE=aarch64-linux-gnu-
```
其中ARCH代表处理器架构，CROSS_COMPILE代表使用的交叉编译工具

#### 设置内核参数

##### 默认设置
此处硬件为树莓派4B，故选择bcm2711
```
make bcm2711_defconfig
```
如果需要其他版本的硬件，则参考[官网](https://www.raspberrypi.com/documentation/computers/linux_kernel.html#cross-compile-the-kernel)

##### 手动设置
如果需要手动设置内核参数，在运行完上面的命令后运行（如果你不知道是否需要手动设置，那跳过即可），如果你选择从原版euler代码构建，则需要进行这一步
```
make menuconfig
```
打开之后在菜单中选择System Type--->Broadcom SoC Support，然后在这个菜单里面把BCM2835 family设置成exclude，这个是树莓派3系和zero的一些支持，此处不需要，如果不取消掉的话后面会编译错误。

配置设置保存完之后会在源码根目录下生成一个.config文件
如果想使用现成的配置文件，把相应文件复制到源码根目录下然后改名为.config即可

#### 编译内核，模块，设备树文件
32位：
```
make zImage modules dtbs -j10
```
64位：
```
make Image modules dtbs -j10
```
Image代表构建内核镜像，modules代表编译内核模块，dtbs代表构建设备树文件。
如果这三个都需要编译的话，也可以直接选择默认方式编译
```
make -j10
```
j后面的参数为线程数，一般设置为cpu核心的1.5倍，cpu的核心数量可以用lscpu命令查看

#### 收集编译结果
如果编译成功，在根目录下会出现vmlinux文件，如果编译不成功则没有
##### 内核模块
```
mkdir output
make INSTALL_MOD_PATH=output/ modules_install
```
在 output/ 文件夹下会生成 lib 文件夹。
##### 内核
32位：
```
cp arch/arm/boot/zImage output/
```
64位:
```
cp arch/arm/boot/Image output/
```
##### 设备树文件和overlay文件
32位:
1. 6.4及以前版本的内核
```
cp arch/arm/boot/dts/bcm271*.dtb output/
mkdir output/overlays
cp arch/arm/boot/dts/overlays/*.dtb* output/overlays
```
2. 6.5及以上版本的内核
```
cp arch/arm/boot/dts/broadcom/bcm271*.dtb output/
mkdir output/overlays
cp arch/arm/boot/dts/overlays/*.dtb* output/overlays
```

64位：
```
cp arch/arm64/boot/dts/broadcom/*.dtb output/
mkdir ${WORKDIR}/output/overlays
cp arch/arm64/boot/dts/overlays/*.dtb* output/overlays/
```
##### 把编译结果移动到上一级目录下
```
cd ..
mkdir output
mv raspberrypi-kernel/output/* output/
sudo rm -r raspberrypi-kernel/output
```

### 测试新编译的内核
使用一个之前刷好32位raspbian镜像或者其他32位系统的SD卡（至于为什么用这个，是因为没有现成的32位openeuler文件系统，只能暂时杂交一下），直接插到Linux主机上，SD会默认挂载其 boot分区和根目录分区。

运行lsblk命令，查看挂载情况，如果sdc代表你的启动媒体（也可能是sdb），如果不知道的话，把sd卡拔了运行一下lsblk，插上等一会儿再运行一遍lsblk，多出来的那个就是。
如果是respbian，应该是这样的结构
```
sdc      
├─sdc1  
└─sdc2   

```
也有的可能是这样的结构
```
sdc      
├─sdc1  
├─sdc2   
└─sdc3   
```
无论是啥样的，根据说明，后面带boot的就是格式化的启动分区，一般是sdc1，另外一个带说明的就是ext4根分区，我的是sdc3，没有说明的那个不用管。

>一般在debian系的发行版里面，有两个分区，sdc1是boot引导分区，sdc2是根文件系统root

>在redhat系里面，有三个分区，一般sdc1是boot引导分区，sdc2是swap交换分区，sdc3是根文件系统root

将这些分区挂载位mnt/boot和/mnt/root，并调整字母和数字以匹配启动分区的位置
```
mkdir mnt
mkdir mnt/boot
mkdir mnt/root
sudo mount /dev/sdc1 mnt/boot
sudo mount /dev/sdc2 mnt/root
```

#### 将内核模块放到根分区里
```
cp -r output/lib/modules mnt/root/lib/
```
#### 将内核放到boot启动分区
32位：
```
cp output/zImage mnt/boot/kernel7l.img
```
64位：
```
cp output/Image mnt/boot/kernel8.img
```

#### 将设备树文件放到boot启动分区

```
cp output/*.dtb mnt/boot/
cp output/overlays/* mnt/boot/overlays
```

#### 设置config.txt
打开树莓派的启动设置文件config.txt
```
vim mnt/boot/config.txt
```
在文件末尾加入
```
arm_64bit=0
kernel=kernel7l.img
```

#### 卸载分区
输入以下命令，将分区卸载，然后将SD卡插入树莓派运行系统
```
sudo umount mnt/boot
sudo umount mnt/root
```

#### 启动树莓派
将SD卡拔下，插入树莓派然后启动，如果能正常启动，说明内核和引导部分编译成功
