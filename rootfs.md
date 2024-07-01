# 完善rootfs
Linux系统就是一个内核加各种程序组成，这些常用的如ls, mkdir, ifconfig命令都是一个个小程序，[构建树莓派镜像](Raspberry-Euler.md)一文中使用的busybox工具是一个工具集合，里面包含了一些最常用的小程序。

如果想选择默认功能更为丰富的rootfs构建工具，可以选择debian系的工具debootstrap，或者使用buildroot
## 设置为可读写文件系统
https://blog.csdn.net/orz365/article/details/9236111

## 添加openssh和sftp

## 添加设备文件
http://www.360doc.com/content/10/0428/11/496343_25245348.shtml
## 添加用户管理

## 添加包管理器