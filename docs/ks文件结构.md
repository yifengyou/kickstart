# ks文件结构


## 参考

* <https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html#chapter-1-introduction>


---


```
The kickstart file is a simple text file, containing a list of items, each identified by a keyword.
```


* ks配置，表现形式是文本，其解析毫无疑问就是文本解析
* 最简单的文本解析：
    - 读取一行，strip()，删除开头结尾空字符（跨行怎么搞？）
    - 提取第一个word作为命令，判断命令是否合法
    - 不合法命令，报错
    - 合法命令，根据具体命令实现形式解析后续参数


```
The Fedora or Red Hat Enterprise Linux installation program also creates a sample kickstart file based on the options that you selected during installation. It is written to the file /root/anaconda-ks.cfg.
```

* 类红帽，或者说所有anaconda作为系统安装程序默认安装好后都会生成/root/anaconda-ks.cfg
* 这个配置文件其实就包括了安装过程中使用的所有选项，当然默认选项不会列出来
* 有些选项参数是必须的，凡是必须的命令绝对会列出来
* 如果ks配置文件不完备会发生什么？系统安装失败，启动失败都有可能，总之，error

```
While not strictly required, there is a natural order for sections that should be followed. Items within the sections do not have to be in a specific order unless otherwise noted. The section order is:
Command section – Refer to Chapter 2 for a list of kickstart options. You must include the required options.
The %packages section – Refer to Chapter 3 for details.
The %pre, %pre-install, %post, %onerror, and %traceback sections – These sections can be in any order and are not required. Refer to Chapter 4, Chapter 5, and Chapter 6 for details.

The %packages, %pre, %pre-install, %post, %onerror, and %traceback sections are all required to be closed with %end
Items that are not required can be omitted.
Omitting any required item will result in the installation program prompting the user for an answer to the related item, just as the user would be prompted during a typical installation. Once the answer is given, the installation will continue unattended unless it finds another missing item.
One installation source command from the list of commands in the method proxy command must be specified for the fully automated kickstart installation. This is required even for Fedora – the closest mirror can’t be chosen by the kickstart file.
Lines starting with a pound sign (#) are treated as comments and are ignored.
If deprecated commands, options, or syntax are used during a kickstart installation, a warning message will be logged to the anaconda log. Since deprecated items are usually removed within a release or two, it makes sense to check the installation log to make sure you haven’t used any of them. When using ksvalidator, deprecated items will cause an error.
```

* 并没有严格要求，各个section的先后顺序，转折，无非就是为了方便查阅，先关section紧挨着
* section大概有两种形式
  - a-z开头的命令 + 参数（可使用 '\' 跨行）
  - %开头的特殊命令
    - **必须用%end结束section**（闭环，四不四很贴切），默认跨行
    - %packages, %pre, %pre-install, %post, %onerror, %traceback
* %packages少不了，其他可选，根据实际需求
* 适配不够完备的话，安装过程中可能弹窗提示用户给出选择，可以继续安装，就是不能全自动，有违初衷
* '#' 开头的都是注释，默认忽略
* 如果用了废弃命令，日志会有警告，不影响过程继续执行




```
Special Notes for Referring to Disks
Traditionally, disks have been referred to throughout Kickstart by a device node name (such as sda). The Linux kernel has moved to a more dynamic method where device names are not guaranteed to be consistent across reboots, so this can complicate usage in Kickstart scripts. To accommodate stable device naming, you can use any item from /dev/disk in place of a device node name. For example, instead of:

part / --fstype=ext4 --onpart=sda1
```

* 特别关注，目标磁盘的锚定方式，如果只有一块盘好办，但是在多盘机器上尽可能明确才更合理

```
part / --fstype=ext4 --onpart=/dev/disk/by-path/pci-0000:00:05.0-scsi-0:0:0:0-part1
part / --fstype=ext4 --onpart=/dev/disk/by-id/ata-ST3160815AS_6RA0C882-part1
```












---
