title: linux命令大全
date: 2016-05-06 13:34:00
tags:
 linux
 order
categories:
 linux
---


主要是为了通过命令的目的或者全名，理解记忆各个命令的功能


<!--more-->

|命令|功能|全名|
|:--:|:--:|:--:|
|startX| 在tty下面开启图形化界面|个人感觉应该是start X Window|
|exit|退出当前登陆的用户||
|date|显示当前日期时间||
|echo|输出一组值|
|cal|日历| calendar|
|bc|简单的计算器，支持（+，-，×，/，^,%）| bash calculator|
|tab|自动补全|
|ctrl+c|种植程序|
|man| 帮助手册|manual|
|/word,?word| 查找，n,N向上向下|
|whatis| 查找某一个命令是什么,等于 man -f|what is|
|info| 功能和man差不多，提供了文件节点的能力|information|
|/usr/share/doc/..|存放了各个命令的说明||
|nano| 文本编辑|//TODO|
|who| 当前有谁在线||
|netstat -a| 当前网络情况|netstat|
|ps -aux| 进程相关|process snapshot|
|shutdown| 关机，重启||
|reboot| 重启||
|halt| 关机||
|poweroff|关机||
|sync|同步|Synchronize|
|fsck| 磁盘扫描检查修复|filesystem scan check and repair|
|ls -al| 列出文件|list|
|chgrp| 更改文件所在组|change group|
|chown|更改文件所有者|change owner|
|chmod|更改文件权限|change mode|
|uname -r| 查看内核版本|//TODO|
|lsb_release -a| 查看distribution信息|//TODO|
|cd| 切换目录|change Directory|
|pwd|显示当前目录|print working Directory|
|mkdir|新建目录| make Directory|
|rmdir|删除一个空目录| remove Directory|
|cp| 拷贝，文件属性不变|copy|
|rm|删除文件|remove|
|mv|移动文件,更改文件名|move|
|basename|取得文件名||
|dirname|取得目录名||
|cat|由第一行开始显示文件类容|concatenate files and print.不是很清楚|
|tac|从最后一行开始显示| cat的倒写
|nl|显示文件并输出行号|number line of files|
|more|一页一页显示||
|less|类似与more，可以向前翻页||
|head|只看头几行||
|tail| 只看结尾几行| |
|od| 以8进制方式读取文件| octal dump files|
|touch|创建新文件，或者更改文件时间| |
|umask|设置或访问新建文件或目录的默认权限|
|which|在当前用户的PATH中寻找命令文件,找不到cd命令，应为是bash内置的||
|whereis|根据数据库，查找特定的文件,-bmsu | where is|
|locate | 根据数据库，查找特定的文件,-ir忽略大小写，使用正则||
|updatedb|更新locate命令依赖的数据库| |
|find|更加强大的查找命令| |
|df | 查看目前挂载的设备| disk filesystem usage|
|du | 查看当前目录下文件与目录的信息| Directory usage|
|dumpe2fs | 查看某一个文件系统的所有情况| dump ext2/ext3/ext4 filesystem information|
|fdisk| 对磁盘进行管理，分区操作等，不能处理>2T的磁盘|manipulate disk partition table|
|mkfs,mke2fs| 磁盘格式化,指定分区信息|make file system|
|fask|磁盘检查,检查文件系统是否出错|file system check|
|badblocks| 检查扇区有没有坏道|search a device for bad blocks|
|mount|挂载设备||
|umount|弹出设备||
|mknod|将设备文件，设置成外部输入设备，存储设备等，设置设备文件的major,minor|make block or character special files|
|e2label|设置设备的卷标|Change the label on an ext2/ext3/ext4 filesystem|
|tune2fs|修改卷标，升级文件系统，读super block数据|adjust tunable filesystem parameters on ext2/ext3/ext4|
|hdparm|设置IDE借口的磁盘的高级参数||
|parted|对高于2TB容量的硬盘进行分区||



**update in 2016-05-17**
