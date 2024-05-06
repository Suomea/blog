```
systemctl start sshd
passwd

ip a

vim /etc/pacman.d/mirrorlist
Server = https://mirrors.cqu.edu.cn/archlinux/$repo/os/$arch

fdisk /dev/sda
/sda1 500M
/sda2 剩余的全部

mkfs.fat -F32 /dev/sda1
mkfs.xfs  /dev/sda2

mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot


pacstrap /mnt base base-devel linux linux-firmware
pacstrap /mnt networkmanager vim sudo

genfstab -U /mnt > /mnt/etc/fstab


arch-chroot /mnt

vim /etc/hostname
suomea98

vim /etc/hosts
127.0.0.1 localhost 
::1 localhost

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

hwclock --systohc

vim /etc/locale.gen

locale-gen
echo 'LANG=en_US.UTF-8'  > /etc/locale.conf

passwd

pacman -S amd-ucode

pacman -S grub efibootmgr

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH

grub-mkconfig -o /boot/grub/grub.cfg

exit
umount -R /mnt
shutdown now
```

```
启动网络，sshd，配置 root 允许密码登录

pacman -Syu

vim /etc/pacman.conf

useradd -m -G wheel -s /bin/bash suomea

pacman -S plasma-meta konsole dolphin

systemctl enable sddm
systemctl start sddm

sudo pacman -S sof-firmware alsa-firmware alsa-ucm-conf 
# 声音固件

sudo pacman -S ntfs-3g 
# 使系统可以识别 NTFS 格式的硬盘

sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei 
# 安装几个开源中文字体。一般装上文泉驿就能解决大多 wine 应用中文方块的问题

sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra 
# 安装谷歌开源字体及表情

sudo pacman -S firefox chromium 
# 安装常用的火狐、chromium 浏览器

sudo pacman -S ark 
# 压缩软件。在 dolphin 中可用右键解压压缩包

sudo pacman -S packagekit-qt6 packagekit appstream-qt appstream 
# 确保 Discover（软件中心）可用，需重启

sudo pacman -S gwenview 
# 图片查看器

sudo pacman -S steam 
# 游戏商店。稍后看完显卡驱动章节再使用

sudo pacman -S fcitx5-im
# 输入法基础包组
sudo pacman -S fcitx5-chinese-addons 
# 官方中文输入引擎
sudo pacman -S fcitx5-material-color 
# 输入法主题
```