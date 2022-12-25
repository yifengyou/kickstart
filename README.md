# anaconda kickstart语法

```
Something I hope you know before go into the coding~
First, please watch or star this repo, I'll be more happy if you follow me.
Bug report, questions and discussion are welcome, you can post an issue or pull a request.
```

* Anaconda uses kickstart to parse and generate kickstart files.
* Pykickstart is a Python 2/3 library consisting of a data representation of kickstart files, a parser to read files into that representation, and a writer to generate kickstart files.


## 相关站点

* pykickstart源码：<https://github.com/pykickstart/pykickstart>
* pykickstart文档：<https://pykickstart.readthedocs.io/>
* anaconda源码：<https://github.com/rhinstaller/anaconda>
* anaconda文档：<https://anaconda-installer.readthedocs.io/>


## kickstart介绍

```
Many system administrators would prefer to use an automated installation method
to install Fedora or Red Hat Enterprise Linux on their machines.
To answer this need, Red Hat created the kickstart installation method.
Using kickstart, a system administrator can create a single file containing the
answers to all the questions that would normally be asked during a typical installation.

Kickstart files can be kept on a server system and read by individual computers
during the installation. This installation method can support the use of a
single kickstart file to install Fedora or Red Hat Enterprise Linux on multiple
machines, making it ideal for network and system administrators.
```


## 目录

* [ks文件是如何生成的](docs/ks文件是如何生成的.md)
* [ks文件结构](docs/ks文件结构.md)
* [ks生成工具-system-config-kickstart](docs/ks生成工具-system-config-kickstart.md)
* [kickstart命令详解](docs/kickstart命令详解.md)
    * [模式选择](docs/kickstart命令详解/模式选择.md)
    * [firstboot行为](docs/kickstart命令详解/firstboot行为.md)
    * [防火墙行为](docs/kickstart命令详解/防火墙行为.md)



## ks配置样本


参看sample目录

```
sample/
├── docker
│   └── anaconda-ks_xxx.cfg
├── iso
│   └── anaconda-ks_xxx.cfg
└── versions.dat
    └── anaconda-ks_xxx.cfg
```










---

<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [kickstart重点语法注释](#kickstart重点语法注释)   
      - [显示模式（install mode）](#显示模式（install-mode）)   
      - [键盘布局（Keyboard layouts）](#键盘布局（keyboard-layouts）)   
      - [网络信息（Network information）](#网络信息（network-information）)   
      - [安装结束后是否重启（Reboot after installation）](#安装结束后是否重启（reboot-after-installation）)   
      - [仓库配置（repository config）](#仓库配置（repository-config）)   
      - [root密码（Root password）](#root密码（root-password）)   
      - [语言配置（language configuration）](#语言配置（language-configuration）)   
      - [时间源配置（time source configuration）](#时间源配置（time-source-configuration）)   
      - [时区选择（time configuration）](#时区选择（time-configuration）)   
      - [使用网络安装（Use network installation）](#使用网络安装（use-network-installation）)   
      - [启动引导配置（System bootloader configuration）](#启动引导配置（system-bootloader-configuration）)   
      - [防火墙配置（firwall configuration）](#防火墙配置（firwall-configuration）)   
      - [selinux配置（selinux configuration）](#selinux配置（selinux-configuration）)   
      - [自动分区（auto partition）](#自动分区（auto-partition）)   
      - [清空MBR分区表（Clear the Master Boot Record）](#清空mbr分区表（clear-the-master-boot-record）)   
      - [删除分区（Partition clearing information）](#删除分区（partition-clearing-information）)   
      - [附加脚本（post script）](#附加脚本（post-script）)   
      - [安装包配置（packages installation configuration）](#安装包配置（packages-installation-configuration）)   
      - [初次启动是否运行初始化程序（first boot）](#初次启动是否运行初始化程序（first-boot）)   
      - [跳过图形界面配置（skip x server configuration）](#跳过图形界面配置（skip-x-server-configuration）)   
      - [忽略特定磁盘（skip disk）](#忽略特定磁盘（skip-disk）)   
      - [是否使用CDROM（using cdrom）](#是否使用cdrom（using-cdrom）)   
      - [系统服务配置（service configuration）](#系统服务配置（service-configuration）)   

<!-- /MDTOC -->

## kickstart重点语法注释

### 显示模式（install mode）

> pykickstart/commands/displaymode.py

只有三种安装模式：cmdline、graphical、text，三种模式必选其一

```
# Use text mode install
text

# Use graphical install
graphical


cmdline
```

这里名称有点迷惑人，会让人怀疑是不是要安装图形界面，其实不然，

此处仅仅是安装时候的模式，实际安装的内容是在%package中指定


### 键盘布局（Keyboard layouts）

> pykickstart/commands/keyboard.py

```
# Keyboard layouts
keyboard 'us'

# Keyboard layouts
keyboard --xlayouts='us'
```

### 网络信息（Network information）

> pykickstart/commands/network.py

这里是配置目标系统的网络而非anaconda的网络

```
# Network information
network --bootproto=dhcp --device=link --activate

network --bootproto=dhcp --device=link --activate --onboot=on

network --hostname=localhost.localdomain
```


### 安装结束后是否重启（Reboot after installation）

> pykickstart/commands/reboot.py

```
# Reboot after installation
reboot

# Shutdown after installation
shutdown
```

### 仓库配置（repository config）

> pykickstart/commands/repo.py


```
repo --name="koji-override-0" --baseurl=https://kojipkgs.fedoraproject.org/compose/36/latest-Fedora-36/compose/Everything/x86_64/os/
repo --name="koji-override-1" --baseurl=https://kojipkgs.fedoraproject.org/compose/updates/f36-updates/compose/Everything/x86_64/os/
```


### root密码（Root password）

> pykickstart/commands/rootpw.py

```
# Root password
rootpw --iscrypted --lock locked
```

### 语言配置（language configuration）

> pykickstart/commands/lang.py

```
# System language
lang en_US.UTF-8
```


### 时间源配置（time source configuration）

> pykickstart/commands/timesource.py

```
timesource --ntp-disable

timesource --nts

timesource --ntp-server=%s

timesource --ntp-pool=%s
```

### 时区选择（time configuration）

> pykickstart/commands/timezone.py

```
# System timezone
timezone Etc/UTC --utc

timezone --utc --nontp UTC
```

### 使用网络安装（Use network installation）

> pykickstart/commands/url.py

```
# Use network installation
url --url="https://kojipkgs.fedoraproject.org/compose/36/latest-Fedora-36/compose/Everything/x86_64/os/"
```

### 启动引导配置（System bootloader configuration）

> pykickstart/commands/bootloader.py

```
# System bootloader configuration
bootloader --disabled       # 跳过系统引导程序安装、配置，常用于docker

# System bootloader configuration
bootloader --location=mbr --timeout=1 --boot-drive=vda
```

### 防火墙配置（firwall configuration）

> pykickstart/commands/firewalld.py

```
firewall --disabled
```

### selinux配置（selinux configuration）

> pykickstart/commands/selinux.py

```
selinux --disabled
```


### 自动分区（auto partition）

> pykickstart/commands/autopart.py

```
autopart --type=plain --nohome --noboot --noswap
```

### 清空MBR分区表（Clear the Master Boot Record）

> pykickstart/commands/zerombr.py

```
# Clear the Master Boot Record
zerombr
```

### 删除分区（Partition clearing information）

> pykickstart/commands/clearpart.py

```
# Partition clearing information
clearpart --all
```

### 附加脚本（post script）

> pykickstart/parser.py

```
%post --logfile=/root/anaconda-post.log --erroronfail
set -eux

# Set install langs macro so that new rpms that get installed will
# only install langs that we limit it to.
LANG="en_US"
echo "%_install_langs $LANG" > /etc/rpm/macros.image-language-conf

# https://bugzilla.redhat.com/show_bug.cgi?id=1727489
echo 'LANG="C.UTF-8"' >  /etc/locale.conf

# https://bugzilla.redhat.com/show_bug.cgi?id=1400682
echo "Import RPM GPG key"
releasever=$(rpm --eval '%{?fedora}')

# When building ELN containers, we don't have the %{fedora} macro
if [ -z $releasever ]; then
  releasever=eln
fi

rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-primary

echo "# fstab intentionally empty for containers" > /etc/fstab

# Remove machine-id on pre generated images
rm -f /etc/machine-id
touch /etc/machine-id
```

```
%post --logfile=/root/anaconda-post.log --erroronfail
# remove some extraneous files
rm -rf /var/cache/dnf/*
rm -rf /tmp/*

# https://pagure.io/atomic-wg/issue/308
printf "tsflags=nodocs\n" >>/etc/dnf/dnf.conf


# https://bugzilla.redhat.com/show_bug.cgi?id=1343138
# Fix /run/lock breakage since it's not tmpfs in docker
# This unmounts /run (tmpfs) and then recreates the files
# in the /run directory on the root filesystem of the container
#
# We ignore the return code of the systemd-tmpfiles command because
# at this point we have already removed the /etc/machine-id and all
# tmpfiles lines with %m in them will fail and cause a bad return
# code. Example failure:
#   [/usr/lib/tmpfiles.d/systemd.conf:26] Failed to replace specifiers: /run/log/journal/%m
#
umount /run
rm -f /run/nologin # https://pagure.io/atomic-wg/issue/316

# Final pruning
rm -rfv /var/cache/* /var/log/* /tmp/*

%end
```


```
%post --nochroot --logfile=/mnt/sysimage/root/anaconda-post-nochroot.log --erroronfail
set -eux

# See: https://bugzilla.redhat.com/show_bug.cgi?id=1051816
# NOTE: run this in nochroot because "find" does not exist in chroot
KEEPLANG=en_US
for dir in locale i18n; do
    find /mnt/sysimage/usr/share/${dir} -mindepth  1 -maxdepth 1 -type d -not \( -name "${KEEPLANG}" -o -name POSIX \) -exec rm -rfv {} +
done

%end
```

### 安装包配置（packages installation configuration）

> pykickstart/parser.py

```
%packages --excludedocs --nocore --inst-langs=en --exclude-weakdeps
bash
coreutils
dnf
dnf-yum
fedora-release-container
fedora-repos-modular
glibc-minimal-langpack
rootfiles
rpm
sudo
tar
util-linux-core
vim-minimal
-dosfstools
-e2fsprogs
-fuse-libs
-glibc-langpack-en
-gnupg2-smime
-grubby
-kernel
-langpacks-en
-langpacks-en_GB
-libss
-pinentry
-shared-mime-info
-sssd-client
-trousers
-util-linux
-xkeyboard-config

%end
```

### 初次启动是否运行初始化程序（first boot）

> pykickstart/commands/firstboot.py

初次启动是否执行```initial-setup```（需安装），默认是不执行

```
firstboot --disable
firstboot --enable
firstboot --reconfig
```

### 跳过图形界面配置（skip x server configuration）

> pykickstart/commands/skipx.py

如果没有图形界面相关组件，默认就是跳过。

```
# Do not configure the X Window System
skipx
```

### 忽略特定磁盘（skip disk）

> pykickstart/commands/ignoredisk.py

```
ignoredisk --only-use=vda
```


### 是否使用CDROM（using cdrom）

> pykickstart/commands/cdrom.py


```
# Use CDROM installation media
cdrom
```


### 系统服务配置（service configuration）

> pykickstart/commands/services.py

```
# System services
services --enabled="sshd,NetworkManager,chronyd,initial-setup"
```















---















---
