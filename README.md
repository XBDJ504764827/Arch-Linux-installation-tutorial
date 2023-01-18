# Arch-Linux-installation-tutorial

Arch linux官网:https://archlinux.org/  
Arch linux官方Wiki地址:https://wiki.archlinux.org/  

## 1.下载ios

Arch linux官网镜像下载地址:https://archlinux.org/download/  
清华源镜像下载地址:https://mirrors.tuna.tsinghua.edu.cn/archlinux/  
阿里巴巴镜像下载地址:https://mirrors.aliyun.com/archlinux/?spm=a2c6h.13651104.0.0.9c8a3f47awkFbW  

## 2.安装
如果是虚拟机或者是同一网段下安装其他机器建议使用ssh安装，首先在需要给Arch设置个root密码，同网段下使用命令行
`ssh@root192.168.xx.xx`进入之后可以进行下一步。

## 3.Arch联网(有网络可跳过)
Arch默认是会被上级路由器dhcp分配地址自动联网的,如果没有就需要手动联网。  
输入以下命令
```
iwctl
```
```
devict list
```
```
station <name> scan
```
```
station <name> get-networks
```
```
station <name> connect <Network name>
```
输入wifi密码之后推出iwd模式再使用
```
ping www.baidu.com
```
测试是否连接至互联网  

## 4.验证启动模式是否为UEFI  
```
ls /sys/firmware/efi/efivars
```
输出如下就可以进行下一步  
![image](https://user-images.githubusercontent.com/94089248/213171614-a9844541-bda9-4e37-ab5b-4fef90574268.png)

## 同步时区(可跳过)
```
timedatectl set-ntp true
```

## 磁盘分区(重要)
```
lsblk
```
![image](https://user-images.githubusercontent.com/94089248/213172577-4390179e-082a-42f3-a481-0dc9e2d82d65.png)  
我需要的是sda这个分区，不同硬盘可能名字不同根据自己事件情况而定。  
```
gdisk /dev/<name>               //<name>为硬盘名
```
依次输入`x` `z` `y` `y`进行分区。  
分区工具较多推荐使用`cgdisk`,也可以选择自己熟悉的分区工具。
```
cgdisk /dev/<name>
```
首先新建boot分区下方选择NEW新建分区，First sector默认即可，size根据情况而定建议使用1024Mib,Hex code不清楚可以依次输入`L``efi`,将EFI前填入即可,name设置为boot。  
新建sawp交换分区First sector默认，size根据可根据下图设置大小，Hex code同上依次输入`L``swap`,将Linux swap前数字输入，name设置为swap。  
![image](https://user-images.githubusercontent.com/94089248/213176387-a6ef0d22-86aa-486c-a35a-710be4f159e6.png)  
新建root分区First sector默认,size建议64G,Hex code默认，name设置为root。  
新建home分区(可与root分区合并，不建议)First sector默认，size默认，Hex code默认，name设置为home。  

完成图  
![image](https://user-images.githubusercontent.com/94089248/213177176-6d85b9c0-98b2-4deb-a1ee-cdc3c6dd0bc1.png)  
完成之后选择下方Write回车之后输入yes就可以退出分区工具了。  

格式化各磁盘  
```
mkfs.fat -F32 /dev/sda1      //boot分区(sda1根据实际情况更改为自己的硬盘名称)
```
```
mkswap /dev/sda2      //swap分区
```
```
swapon /dev/sda2      //启用swap分区
```
```
mkfs.ext4 /dev/sda3   //root分区
```
```
mkfs.ext4 /dev/sda4   //home分区
```
上述硬盘名称请自行更改为自己硬盘名称。  

挂载分区(没有的分区需要创建)  
```
mount /dev/sda3 /mnt
```
```
mkdir /mnt/boot
```
```
mount /dev/sda1 /mnt/boot
```
```
mkdir /mnt/home
```
```
mount /dev/sda4 /mnt/home
```
完成之后可以对照下图看看是否正确  
![image](https://user-images.githubusercontent.com/94089248/213180105-e8747dea-f74e-4a2b-af1b-7ce103f804fa.png)  

## 设置pacman服务器
使用喜欢的文本编辑器打开`/etc/pacman.d/mirrorlist`文件,在首行添加以下源  
清华源:  
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch  
阿里源:  
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch  
163源:  
Server = http://mirrors.163.com/archlinux/$repo/os/i686  
