title: linux杂记
date: 2016-05-06 13:31:06
tags: [linux]
categories: [linux]
---

linux 中的一些琐碎知识点
<!--more-->
# 目录标识：
|目录|意思|
|:--:|:--:|
|.|当前目录|
|..|上一层目录|
|-|前一个工作目录|
|~|用户所在主目录|
|~account|account所在主目录|

# 文件属性
> ** -rw-r--r-- l username group size last modify time filename **
> 这里的文件时间，指的是mtime,所谓的文件时间还包括 ctime, actime

### 系统挂载
1. 根目录必须挂载，而且一定要先于其他mount point被挂载进来
2. 其他挂载点必须为已经新建的目录
3. 所有的挂载点/分区，在同一时间只能挂载一次
4. 如果要卸载，必须先将工作目录，移除到挂载点之外。
