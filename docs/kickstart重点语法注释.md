<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [kickstart重点语法注释](#kickstart重点语法注释)   
      - [install mode](#install-mode)   
      - [Keyboard layouts](#keyboard-layouts)   
      - [Network information](#network-information)   
      - [Reboot after installation](#reboot-after-installation)   
      - [repository config](#repository-config)   
      - [Root password](#root-password)   
      - [time configuration](#time-configuration)   
      - [Use network installation](#use-network-installation)   
      - [System bootloader configuration](#system-bootloader-configuration)   
      - [Clear the Master Boot Record](#clear-the-master-boot-record)   
      - [Partition clearing information](#partition-clearing-information)   
      - [post script](#post-script)   
      - [packages installation configuration](#packages-installation-configuration)   

<!-- /MDTOC -->

## kickstart重点语法注释

### install mode

```
text

graphical
```

### Keyboard layouts

```
keyboard 'us'
```

### Network information

```
network  --bootproto=dhcp --device=link --activate
```


### Reboot after installation

```
reboot
```

### repository config

```
repo --name="koji-override-0" --baseurl=https://kojipkgs.fedoraproject.org/compose/36/latest-Fedora-36/compose/Everything/x86_64/os/
repo --name="koji-override-1" --baseurl=https://kojipkgs.fedoraproject.org/compose/updates/f36-updates/compose/Everything/x86_64/os/
```


### Root password

```
rootpw --iscrypted --lock locked
```

### time configuration

```
timesource --ntp-disable
timezone Etc/UTC --utc
```

### Use network installation

```
url --url="https://kojipkgs.fedoraproject.org/compose/36/latest-Fedora-36/compose/Everything/x86_64/os/"
```

### System bootloader configuration

```
bootloader --disabled
autopart --type=plain --nohome --noboot --noswap
```

### Clear the Master Boot Record

```
zerombr
```

### Partition clearing information

```
clearpart --all
```

### post script

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

### packages installation configuration

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
