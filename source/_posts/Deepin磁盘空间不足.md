---
title: Deepin磁盘空间不足
date: 2017-09-22 22:52:52
tags: [Linux,系统]
categories: [Deepin]
toc: true
cover: '/images/categories/deepin.jpeg'
---
# 简介
早上打开电脑进入Deepin系统，系统右上角突然提示磁盘空间不足！！！ (啊啊啊！没图啊，问题已经修复了，回来总结的，自己脑补吧)
然后`df -l`了一下，发现果然有问题，
```bash
    df -l
```
<img src="/images/deepin/df-l.png">

自己研究了一番，没有进展，于是乎想起了我的运维哥们。在他的指点下果然发现了问题所在：
发现`/home`这个目录占用了50多G的空间，Deepin我用就分给了100G的空间，以为够用的啊（崩溃。。。）
# 分区
走亲访友里一圈最终获得解决方法：将`/home`目录重新挂在到一个空分区上（也就是上图的sda8了，看到了吧）。

首先创建一个分区：
```bash
    sudo fdisk /dev/sda8     #/dev/sda8就是你指定要修改的磁盘区块
```
然后根据提示来就好

第二部，格式化分区
```bash
    sudo mkfs -t ext4 /dev/sda8
```
然后再执行下一次`df -l`
```bash
    df -l
```
<img src="/images/deepin/blkid.png">
可以看到新建分区的UUID和TYPE。这里的type就是分区格式，可以看到`/dev/sda8`是`ext4`格式的

# 复制
到这里就需要备份`/home`目录下的文件了，为了保证重新挂在`/home`分区，系统能够正常运行，需要将`/home`下的所有文件（包括隐藏文件）都拷贝到新的分区中（即用来代替原有home分区的sda8）

```bash
    sudo mkdir /mnt/home
    sudo mount /dev/sda8 /mnt/home/             #首先将分区挂在到一个地方，方便复制文件

    cp -a /home/* /mnt/home/                 #这里就等吧，看你有多少文件了
```
修改`/etc/fstab`
```bash
    sudo vim /etc/fstab
```
添加默认挂在目录`/home`(`/dev/sda8`就是后添加进去的，挂载了`/home`)
```bash
# /dev/sda7
UUID=3abf9ed4-1eeb-49e2-a381-1fc782dac734	/         	ext4      	rw,relatime,data=ordered	0 1
# /dev/sda8
UUID=38935abe-a24d-4d5f-96b7-0cf4c47e95b4	/home		ext4		rw,relatime,data=ordered	0 1

```
保存退出，然后重启（这里要看看系统是否能正常重启，如果失败可能就要退回去了。。。）

*注意：这里要重启大概3次*

重启成功之后，检查一切是否正常。正常将`/etc/fstab`的第二个挂在注释掉，再重启：
```bash
sudo vim /etc/fstab

-------------------------------------------------------------------------------
# /dev/sda7
UUID=3abf9ed4-1eeb-49e2-a381-1fc782dac734	/         	ext4      	rw,relatime,data=ordered	0 1
# /dev/sda8
#UUID=38935abe-a24d-4d5f-96b7-0cf4c47e95b4	/home		ext4		rw,relatime,data=ordered	0 1
-------------------------------------------------------------------------------

sudo reboot
```

# 删除原来的/home文件夹
为什么要删除呢（废话，不删除，根目录磁盘空间不还是满的嘛！）

这次重启之后以`root`用户身份登录系统（非图形界面），然后执行一下命令
```bash
    rm -rf /home            #目前这个/home还是老的home
```

# 挂载
```bash
sudo vim /etc/fstab     #再把注释放开，这次真正让/home指向/dev/sda8

-------------------------------------------------------------------------------
# /dev/sda7
UUID=3abf9ed4-1eeb-49e2-a381-1fc782dac734	/         	ext4      	rw,relatime,data=ordered	0 1
# /dev/sda8
UUID=38935abe-a24d-4d5f-96b7-0cf4c47e95b4	/home		ext4		rw,relatime,data=ordered	0 1
-------------------------------------------------------------------------------

sudo reboot
```
好了，就此终于结束了！


---------------------------------分割线---------------------------------------
## BUG BUG BUG
（已哭晕在厕所。。。）
重启之后好久才发下Deepin的启动器打不开了[\大哭]



