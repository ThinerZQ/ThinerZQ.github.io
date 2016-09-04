title: mysql win64 5.7.9 zip包安装日记
date: 2016-09-01 10:52:20
tags:
 mysql
categories:
 mysql
---

# 官网下载win64 zip包
找到合适的目录解压
# 配置my-default.ini
> basedir = D:\Study\MySQL\mysql-5.7.14-server-win64
 datadir = D:\Study\MySQL\mysql-5.7.14-server-win64\data
 port = 3306

 #管理员方式打开命令行，进入mysql home 目录下的bin目录

执行如下命令：
>1. mysqld --initialize（我使用mysqld --initialize -secure生成的data目录下的文件不完整）
在mysql的根目录下自动生成data目录以及其他文件，初始化一个随机密码的root用户
2. mysqld -install/remove
注册mysql服务
3. net start/stop mysql
启动服务

# 关于root的随机密码在那里
windows在data目录下的DESKTOP-M03IEJ5.err文件里面，搜索“2016-09-01T02:15:22.399996Z 1 [Note] A temporary password is generated for root@localhost: -/br>,B;o9gm”这样的一个字符串就能找到了，其他系统也都写在日志里面，找找总会有的

# 最后:mysql -u root -p password 进入mysql 控制台
更改mysql密码
> alter user 'root'@'localhost' identified by 'your password';

that's all
