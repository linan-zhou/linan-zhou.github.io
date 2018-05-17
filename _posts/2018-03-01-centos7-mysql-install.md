---
layout: post
title: 在CentOS7上安装MySQL
tags: [CentOS7, MySQL, Mariadb]
excerpt_separator: <!--more-->
---
本文介绍了在安装了CentOS7的远程VPS上面安装MySQL的过程和一些注意事项
<!--more-->
首先CentOS上常用yum软件包管理器，比较简单方便。而安装MySQL，通常需要安装MySQL-client（前台用来操作MySQL的进程）和MySQL-server（启动在后台的服务端），还有很多推荐安装MySQL-devel包的。
*-devel包主要是一些软件开发所需要的文件，比如MySQL-devel就是MYSQL开发所需要的包和文件，如果只是需要在机器上使用MySQL的话，这个包可以不用安装。
下面安装MySQL-client和MySQL-server

    sudo yum install mysql

输入密码后安装完毕，接着安装mysql-server

    sudo yum install mysql-server

这时却报错：

    No package mysql-server available.
    Error: Nothing to do

也就是说没有这个包
下面用yum list mysql*语句可以查看可以安装的名字带mysql的包都有什么，来确定是否真的没有mysql-server这个包

    [username@host ~]$ yum list mysql*

    Loaded plugins: fastestmirror
    Loading mirror speeds from cached hostfile
     * base: mirror.sfo12.us.leaseweb.net
     * elrepo-kernel: repos.lax-noc.com
     * extras: mirror.hostduplex.com
     * updates: mirrors.syringanetworks.net
    Available Packages
    MySQL-python.x86_64                                                                            1.2.5-1.el7                                                                       base
    mysql-connector-java.noarch                                                                    1:5.1.25-3.el7                                                                    base
    mysql-connector-odbc.x86_64
                                                               5.2.5-7.el7                                                                       base
可以看出，可以装的MySQL相关的软件里真的没有MySQL-server，这是怎么回事呢？经过查询，CentOS7中，把MySQL相关的包都从yum包管理器中移除了，换成了Mariadb
Mariadb是一个可以用来代替MySQL的开源社区版本，维基百科的词条描述：

>MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。

所以在CentOS7中，实际上只有Mariadb的Client和Server可以安装，前面的yum install mysql自动装了Mariadb的Client，可以用yum list installed|grep 关键字 查看已经装过的相关的包
经查看，确实新增了Mariadb相关的软件，而没有Mysql相关的
知道前因后果以后，我们可以直接装Mariadb-server

    sudo yum install mariadb-server

    这下就安装完成了，接下来先启动server：

    systemctl start mariadb

    再启动client：

    [username@host ~]$ mysql -u root
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 2
    Server version: 5.5.56-MariaDB MariaDB Server

    Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    MariaDB [(none)]>

    就可以在客户端使用MySqL啦
