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
保存之后执行
```
pacman -Syy
```

## 安装系统及固件
```
pacstrap -i /mnt linux linux-headers linux-firmware base base-devel nano vim intel-ucode(cpu为AMD的改为amd-ucode)
```
出现选项默认即可。  
安装完成之后需要写入
```
genfstab -U -p /mnt >> /mnt/etc/fstab
```
```
arch-chroot /mnt      //进入系统
```

安装引导程序
```
pacman -S grub efibootmgr
```
```
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=ArchLinux
```
生成grub文件
```
grub-mkconfig -o /boot/grub/grub.cfg
```
## 设置中英文编码格式(可选择在安装完成KDE桌面环境之后更改)
```
nano /etc/locale.gen
```
找到en_US.UTF-8 UTF-8和zh_CN.UTF-8 UTF-8取消前面#号注释。保存之后输入`locale-gen`更新  
为防止英文乱码可以需要设置个全局locale  
```
nano /etc/locale.conf
```
键入`LANG=en_US.UTF-8`保存并退出。  
下面这个代码为终端,窗口系统，显示管理器配置中文化。
```
echo -e "export LANG=zh_CN.UTF-8\nexport LANGUAGE=zh_CN:en_US" > ~/.bashrc && echo -e "export LANG=zh_CN.UTF-8\nexport LANGUAGE=zh_CN:en_US" > ~/.xinitrc && echo -e "export LANG=zh_CN.UTF-8\nexport LANGUAGE=zh_CN:en_US" > ~/.xprofile
```

## 更改电脑名称以及host
```
echo "<name>" > /etc/hostname     //<name>为你想给电脑取的名字
```
```
nano /etc/hosts
```
在下方另起新行  
```
127.0.0.1 localhost
::1       localhost
127.0.1.1 <name>.localdomain  <name>
```
my-arch-linux为我为电脑取的名字,写成下方这个样子就可以了  
![image](https://user-images.githubusercontent.com/94089248/213189785-4932eede-3ad5-4964-ae3c-8ebd6b850a71.png)  

## 添加用户设置权限
首先为root用户设置密码。设置完成之后添加用户`useradd -m -g users -G wheel,storage,power -s /bin/bash <user>     \\<user>为用户名称`  
为普通用户设置密码。为新建用户添加权限`EDITOR=nano visudo`上方nano可改为vim。  
![image](https://user-images.githubusercontent.com/94089248/213198036-57e7493f-3014-43ed-a483-97857f6e177b.png)

## 安装配置systemd启动器
```
bootctl install
```
```
nano /boot/loader/entries/arch.conf
```
```
title My arch linux
linux /vmlinuz-linux
initrd /intel-ucode.img         //AMD改为amd-ucode.img
initrd /initramfs-linux.img
```
```
echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sda3) rw" >> /boot/loader/entries/arch.conf
```
再次打开/boot/loader/entries/arch.cong文件检查。
![image](https://user-images.githubusercontent.com/94089248/213199816-698cabca-8b27-4b63-b6a3-d579accc5d5b.png)  

## 安装基本包
```
pacman -S networkmanager network-manager-applet dialog wpa_supplicant dhcpcd
```
```
pacman -S mtools dosfstools bluez bluez-utils cups xdg-utils xdg-user-dirs alsa-utils pulseaudio pulseaudio-bluetooth reflector openssh htop
```
## 安装显卡驱动(可选)
Arch linux给出的驱动根据实际情况安装
![image](https://user-images.githubusercontent.com/94089248/213201083-d62ae951-026c-4bfe-8397-eb6519ae2c1a.png)  
我是不准备给Arch直通的所以安装个intel的驱动就行了
```
pacman -S xf86-video-intel mesa libva-mesa-driver mesa-vdpau dkms libglvnd
```

到这里就可以退出chroot环境，使用`umont -R /mnt`解除挂载然后重启使用了。

## 安装中文字体
```
sudo pacman -S adobe-source-han-sans-cn-fonts
sudo pacman -S adobe-source-han-serif-cn-fonts
sudo pacman -S noto-fonts-sc
sudo pacman -S wqy-microhei
sudo pacman -S wqy-zenhei
sudo pacman -S wqy-bitmapfont
sudo pacman -S ttf-arphic-ukai
sudo pacman -S ttf-arphic-uming
sudo pacman -S opendesktop-fonts
sudo pacman -S ttf-hannom
```
