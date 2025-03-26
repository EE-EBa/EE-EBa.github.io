---
title: Archlinux usage 
date: 2025-03-26
categories: [archlinux]
tags: [Linux , iwctl ]
---

# issue 1

Cause I not log in my arch for two days, I can not launch KDE service when I start my PC.Meanwhile I can not use tty1~6 to solve this issue. When I arrived tty windows , I can not input any parameters in command line.

## solved method:

Making a archlinux installing USB. Cause we can not into damaged system from tty, so we just into this archlinux by installing USB.

### Solve steps:

#### 1. Making a archlinux installing USB
#### 2. start PC , chose normal mode->chose archlinux install x86
#### 3. connect you internet.

```shell
iwctl
iwctl:station wlan0 scan
iwctl:station wlan0 connect you-wifi
exit
ping www.gnu.org
```

#### 4. synchro your local time

```shell
timedatectl set-ntp true
```

#### 5. Mount system partitioning

first of all , you neet mount bootloader(efi) partitioning,second mount boot partitioning,last mount home.

```shell
mount /dev/nvme0n1p3 /mnt
mount /dev/nvme0n1p1 /mnt/efi
mount /dev/nvme0n1p4 /mnt/home
```

#### 6. Into damaged system

```shell
arch-chroot /mnt
```

#### 7.  reinstall your linux kernel

```shell
pacman -S linux
```
#### 8. update your system

```shell
pacman -Syu
```

#### 9. installing bootloader

Maybe you can see "no boot device",when you reboot you system so you need to reinstall bootloader.

```shell
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```

#### 10. reboot you system

```shell
exit
reboot
```

Issue solved!

# Configuration of git

## 在archlinux中需要安装插件openssh才行

此插件在pacman包中包含有,直接使用下列命令安装即可<br>

```shell
sudo pacman -S openssh
```

## 将本地仓库推送到github的步骤如下:

### 1. 安装好依赖如 openssh
### 2. 在github上创建好token,用于推送代码所用.步骤为点击github头像->setting->developer setting->personal access tokens->token(class)或者beta版本也行
### 3. 在github创建远程仓库
### 4. 创建本地仓库并推送到github

```shell
git init
git add filename #添加文件到版本管理
git commit -m "comment" #提交更改内容
git remote add origin 仓库链接 #创建一个远程仓库连接并且修改别名为origin
git push -u origin main #将本地参考分支main推送到github仓库
```

### 5. 注意在输入最后一条push命令时,需要输入github的账户名和密码,密码并不是真正的github密码,需要输入之前创建的token来代替.


## 使用github-cli进行token管理
在使用gitpush命令时每次都要输入token来验证信息,非常不方便.通过使用工具github-cli可以实现自动存储token<br>
采用下列命令可以进行github-cli安装.<br>

```shell
sudo pacman -S github-cli
```
通过交互式互动,可以简单的实现安装.


# Configuration of stow

## 0. 背景

在安装新系统时,许多软件的配置需要重头配置一边,这会浪费许多时间.<br>
在linux系统下,一般我们安装的软件都存在与home目录下,且其配置文件基本为.name形式隐藏存在的.<br>
因此一般需要使用以下命令才能找到如.vimrc,.zshrc配置文件<br>

```shell
ls .a
```

### 1. 创建git仓库用于保存重要软件的配置

为了方便管理配置文件,我们可以将这类文件复制到home目录下新创建的dotfiles中统一管理.<br>
将dotfiles目录整个上传至github便于装配新机或者更换配置所用<br>

### 2. 使用stow工具来快速还原软件配置

由于我们这里修改了配置文件的路径,因此linux系统是无法找到配置文件的.<br>
因此需要采用工具stow来进行路径映射,让在linux系统在home目录下创建一个类似快捷链接的文件,指向dotfiles目录里的真实配置文件.<br>
常用命令有如下:<br>

```shell
cd ~/dotfiles
stow -t $HOME git
stow -t $HOME tmux
stow -t $HOME zsh
```
如果你的 dotfiles 直接位于 HOME 目录下（~/dotfiles/），那么可以省略 -t $HOME 参数，stow 默认会软链接到上级目录。

### 3. 编写shell脚本一键映射配置文件

```shell

list=(zsh git tmux nvim ranger conda)
for i in ${list[*]}; do
    stow -t $HOME $i || exit -1
done
```

在list 中列出所有要安装的配置文件，执行 install.sh 就可以装上自己需要的配置了。
