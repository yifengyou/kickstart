# ks文件是如何生成的


检索代码你会发现，anaconda中使用了pykickstart组件，一方面用来解析传递给anaconda的kickstart配置，另一方面，就算没有ks配置，anaconda默认也会调用pykickstart生成一份（有且至少一个ks）安装后的ks文件，其配置将放在家目录下

检索kickstart python包你会发现，这里面的很多字串其实就是ks配置里的

```
[root@fedora36 /data/pykickstart.git/pykickstart/commands]# grep -rn "+= \"# "
authconfig.py:41:            retval += "# System authorization information\nauth %s\n" % self.authconfig
authselect.py:37:            retval += "# System authorization information\nauthselect %s\n" % self.authselect
bootloader.py:76:            retval += "# System bootloader configuration\nbootloader"
cdrom.py:36:        retval += "# Use CDROM installation media\ncdrom\n"
clearpart.py:62:        retval += "# Partition clearing information\nclearpart%s%s%s\n" % (clearstr, initstr, drivestr)
displaymode.py:47:            retval += "# Use graphical install\ngraphical\n"
displaymode.py:49:            retval += "# Use text mode install\ntext\n"
eula.py:39:            retval += "# License agreement\n"
firewall.py:77:            retval += "# Firewall configuration\nfirewall --enabled%s%s%s\n" % (extrastr, portstr, truststr)
firewall.py:79:            retval += "# Firewall configuration\nfirewall --disabled\n"
firstboot.py:44:            retval += "# Run the Setup Agent on first boot\nfirstboot --enable\n"
firstboot.py:46:            retval += "# Run the Setup Agent on first boot\nfirstboot --reconfig\n"
harddrive.py:55:        retval += "# Use hard drive installation media\n"
harddrive.py:113:        retval += "# Use hard drive installation media\n"
hmc.py:36:        retval += "# Use installation media via SE/HMC\nhmc\n"
interactive.py:37:            retval += "# Use interactive kickstart installation method\ninteractive\n"
keyboard.py:42:            retval += "# System keyboard\nkeyboard %s\n" % self.keyboard
keyboard.py:83:            retval += "# old format: keyboard %s\n" % self._keyboard
keyboard.py:84:            retval += "# new format:\n"
lang.py:40:            retval += "# System language\nlang %s\n" % self.lang
liveimg.py:50:        retval += "# Use live disk image installation\n"
logging.py:47:            retval += "# Installation logging level\nlogging --level=%s" % self.level
logging.py:92:            retval += "# Installation logging level\nlogging --host=%s" % self.host
mouse.py:49:            retval += "# System mouse\nmouse %s%s\n" % (opts, self.mouse)
nfs.py:49:        retval += "# Use NFS installation media\nnfs --server=%s --dir=%s\n" % (self.server, self.dir)
ostreesetup.py:40:            retval += "# OSTree setup\n"
realm.py:93:            retval += "# Realm or domain membership\n"
reboot.py:41:            retval += "# Reboot after installation\nreboot"
reboot.py:44:            retval += "# Shutdown after installation\nshutdown"
rootpw.py:56:            retval += "# Root password\nrootpw%s %s\n" % (self._getArgsAsStr(), password)
selinux.py:41:        retval += "# SELinux configuration\n"
services.py:48:            retval += "# System services\nservices%s\n" % args
skipx.py:37:            retval += "# Do not configure the X Window System\nskipx\n"
timezone.py:48:            retval += "# System timezone\ntimezone %s %s\n" % (utc, self.timezone)
timezone.py:100:            retval += "# System timezone\ntimezone %s %s\n" % (utc, self.timezone)
timezone.py:122:            retval += "# System timezone\n"
timezone.py:298:            retval += "# System timezone\n"
upgrade.py:44:            retval += "# Upgrade existing installation\nupgrade\n"
upgrade.py:46:            retval += "# Install OS instead of upgrade\ninstall\n"
upgrade.py:84:            retval += "# Upgrade existing installation\nupgrade --root-device=%s\n" % self.root_device
url.py:50:            retval += "# Use network installation\nurl --url=\"%s\"\n" % self.url
url.py:160:        retval += "# Use network installation\n"
url.py:220:        retval += "# Use network installation\n"
zerombr.py:42:            retval += "# Clear the Master Boot Record\nzerombr\n"
zipl.py:74:            retval += "# ZIPL configuration\nzipl"
[root@fedora36 /data/pykickstart.git/pykickstart/commands]#


```
