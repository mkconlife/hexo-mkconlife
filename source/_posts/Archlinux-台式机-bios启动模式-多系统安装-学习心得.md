---
title: Archlinux 台式机 bios启动模式 多系统安装 学习心得
cover: [/img/mydesktop.png]
keywords: Archlinux,0基础
banner: 
  bannerText: Hi my new friend!
abbrlink: fc27d993
date: 2023-04-29 18:28:47
---
# Archlinux 台式机 bios启动 多系统安装 学习心得
丑话说在前头，不愿意折腾的还是换个linux发行版吧，要是你喜欢pacman和AUR，可以尝试去用`Manjaro`.
本次使用的电脑已经很老了，但是iso是新的，不保证能在你的电脑上完全安装成功，本文只是提供一个思路罢了.
废话少说(虽然已经说不少了)，现在步入正题
## 一、准备
若有`boot`分区，请确保其分区的大小超过1G，小于1G可以使用diskgenius进行分区大小调整(食用方法找！度！娘！)
你需要了解你的cpu是哪一家厂商生产的和你windows所在的分区号(如果你电脑磁盘每个分区分的存储空间都不一样，只需记住你windows所在磁盘的大小即可，如果你电脑系统没有装在逻辑分区上，可以直接用`磁盘管理`或者`DiskGenius`从左往右数，记下你的win系统(或者其他的)在第几个上，若装在逻辑分区，那么将你数的那个数加一，还有一种比较暴力的方法，在你的C盘根目录下创建 一个方便找的文件，然后在启动`archiso`后输入

```
mount /dev/sda1 /mnt
cd /mnt
ls
cd ..
umount /mnt
mount /dev/sda2 /mnt
```
...........
直到找到那个文件为止，记下你的分区号`(/dev/sdax)`
准备一个可启动的U盘(或者一个已经被root了的手机和一根数据线)(U盘建议至少`8G`，虽然iso镜像没有这么大，但是可以避免遇到那些劣质U盘(?))，iso镜像下载可以选择清华源和163源，没有U盘的可以把镜像传到已经root好了的手机上，然后下载`DriverDroid`，食用方式去找度娘QwQ
注意:**致`ventoy`用户:如果你使用的是ventoy启动方式，请把ios镜像放到一个文件都没有的u盘分区，若找不到这个分区，请用电脑自带的磁盘管理或者DiskGenuis赋予其盘符**，到ventoy启动界面后选择你下载好的镜像回车，然后请务必选择“以grub2模式启动”，否则将会出现致命错误(你不一定抽到什么错误:D，有一个好像是关于解压文件的错误[他的英文直译过来后是解压缩文件被腐蚀])虽然他的grub2启动项标题都带着uefi，但是不用担心，它不会影响到我们的安装过程，选择第一个启动即可(启动速度可能很慢)
![内核错误](/img/kernelpanic.jpg)
## 二、安装
此时应进入到`archiso`

### 1.联网
家里只有台式机，所以不提供无线网络联网方式的安装教程
台式机的联网很简单，输入`dhcpcd`回车即可

### 2.分区
我从接触archlinux起分区就一直用的是`cfdisk`
```
cfdisk
```
若给剩余空间分区，用方向键上下选中`free space`，然后用方向键左右选中`new`，回车，输入分区大小(要是创建逻辑分区请直接回车)，回车，尽量选择`primary`(主分区)，如果超过四个主分区可以选择`extend`(逻辑分区)(若是选择extend请在创建好后的分区下面的`freespace`重新新建分区)，然后方向键左右选中`Write`,回车，输入yes回车然后记下分区号，建议再创建一个`swap`(虚拟内存，就是拿你的存储空间当做内存用)，4G左右即可(因机器而异)，然后方向键左右选中`Write`，回车然后记下分区号,然后方向键左右移动到`Quit`回车
运行
```
mkfs.ext4 你要装系统的分区号
```
然后输入y回车
```
mkswap swap的分区号
swapon swap的分区号
```
### 3.安装
**3.1 挂载分区**
```
mount 你要装系统的分区号 /mnt
```
如果有boot分区，运行
```
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```
**3.2 接下来换镜像站**
```
echo 'Server = https://mirror.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch' > /etc/pacman.d/mirrorlist
```
**3.3 安装系统**
```
pacman -Sy
pacstrap /mnt base base-devel linux linux-firmware vim dhcpcd grub os-prober ntfs-3g net-tools
```
**3.4 生成文件系统描述表**
```
genfstab -U /mnt >> /mnt/etc/fstab
```
**3.5 进入刚安装好的系统**
```
arch-chroot /mnt
```
**3.5-1 设置密码(root用户)**
```
passwd
```
注意，在linux终端输入密码是不可见的，若存在数字密码请确保你的数字锁定已经打开
**3.5-2 配置时区**
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
cat /etc/localtime
```
出现乱码则成功
![/etc/localtime](/img/localtime.png)
```
hwclock --systohc
```
**3.5-3 本地化**
```
vim /etc/locale.gen
```
用方向键下找`en_US UTF-8 UTF-8`和`zh_CN UTF-8 UTF-8`
输入i
删去前面的`#`
按esc，输入`:wq!`
```
locale-gen
echo LANG=en_US.UTF-8 > locale.conf
```
**3.5-4网络配置**
```
echo 起个你的主机名 > /etc/hostname
vim /etc/hosts
```
输入i
输入
```
127.0.0.1(这里按Tab键)localhost
::1(这里按Tab键)(这里按Tab键)localhost
127.0.1.1(这里按Tab键)主机名.localdomain(这里按Tab键)主机名
```
![Hosts Example](/img/hosts.png)
按esc输入:wq!以保存并退出
**3.5-5安装微码(不安装连引导页面都进不去，别问我怎么知道的:D)**
```
pacman -S intel-ucode(或者amd-ucode，根据你的cpu型号来定)
```
**3.5-6部署grub**
```
grub-install ––target=i386-pc /dev/sda
```
如果你的电脑上还存在`Windows`
那么键入
```
vim /etc/default/grub
```
取消下面这一行的注释，如果没有相应注释的话就在文件末尾添加上：
`GRUB_DISABLE_OS_PROBER=false`

配置grub文件
```
grub-mkconfig -o /boot/grub/grub.cfg
exit
reboot
```
**3.5-e 网络不可用的解决办法**
注意，此时的用户名是root
如果登上archlinux后发现无法联网
输入
```
echo interface = eth0 > /etc/rc.conf
dhcpcd
systemctl enable dhcpcd
```
即可解决
有时系统滚完后可能会再次无法联网，此时在执行完第一行命令后执行
```
touch /etc/rc.conf 
```
**3.5-7 创建用户**
在linux中使用root用户会带来诸多不便，所以我们要创建一个普通用户
```
useradd -m -G wheel -s /bin/bash 你的用户名
```
```
vim /etc/sudoers
```
输入i
取消`%wheel ALL=(ALL:ALL) ALL`的注释
然后输入i键入 `你的用户名 ALL=(ALL) ALL`
按esc输入`:wq!`以保存并退出
```
passwd 你的用户名
```
然后输入你想要设置的密码
**3.6 重启**
```
exit
reboot
```
至此，一个基本的Archlinux就安装完了:D
感谢你看到这里

参考链接
1. [ArchlinuxWiki-GRUB](https://wiki.archlinuxcn.org/wiki/GRUB#%E6%8E%A2%E6%B5%8B%E5%85%B6%E4%BB%96%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F)
2. [ArchlinuxWiki-安装教程](https://wiki.archlinuxcn.org/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)
3. [Archlinux Bios安装](https://blog.csdn.net/XM_89/article/details/121364518)
4. [Archlinux无法上网](https://blog.csdn.net/weixin_35315373/article/details/116969992)

