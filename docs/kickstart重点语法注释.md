<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [kickstart重点语法注释](#kickstart重点语法注释)   
      - [显示模式（install mode）](#显示模式（install-mode）)   
      - [键盘布局（Keyboard layouts）](#键盘布局（keyboard-layouts）)   
      - [网络信息（Network information）](#网络信息（network-information）)   
      - [安装结束后是否重启（Reboot after installation）](#安装结束后是否重启（reboot-after-installation）)   
      - [仓库配置（repository config）](#仓库配置（repository-config）)   
      - [root密码（Root password）](#root密码（root-password）)   
      - [时间源配置（time source configuration）](#时间源配置（time-source-configuration）)   
      - [时区选择（time configuration）](#时区选择（time-configuration）)   
      - [Use network installation](#use-network-installation)   
      - [System bootloader configuration](#system-bootloader-configuration)   
      - [自动分区（auto partition）](#自动分区（auto-partition）)   
      - [清空MBR分区表（Clear the Master Boot Record）](#清空mbr分区表（clear-the-master-boot-record）)   
      - [删除分区（Partition clearing information）](#删除分区（partition-clearing-information）)   
      - [附加脚本（post script）](#附加脚本（post-script）)   
      - [安装包配置（packages installation configuration）](#安装包配置（packages-installation-configuration）)   

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
```

### 网络信息（Network information）

> pykickstart/commands/network.py

```
# Network information
network  --bootproto=dhcp --device=link --activate
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
```

### Use network installation

> pykickstart/commands/url.py

```
# Use network installation
url --url="https://kojipkgs.fedoraproject.org/compose/36/latest-Fedora-36/compose/Everything/x86_64/os/"
```

### System bootloader configuration

> pykickstart/commands/bootloader.py

```
# System bootloader configuration
bootloader --disabled
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
