---
layout: post
title: 使用远程主机之前的一些的基本安全建议
tags: [VPS, sudo, ssh]
excerpt_separator: <!--more-->
---
当拿到一台新的远程主机的时候，在开始使用之间，有一些必要的安全方面的操作需要进行。本文以CentOS7为例，进行讲解。
<!--more-->
###一.创建新用户，尽量不用root直接操作
    首先创建一个新用户，尽量不要直接用root用户登录操作。这是因为，使用非root用户登录的时候，在进行一些高危操作的时候会有权限提示，二次确认和用sudo语句执行这些高危操作的时候能够降低误操作的概率。
    首先在terminal中用root登录远程主机：
    >ssh root@XXX.XXX.XXX.XXX（远程主机IP地址） -p 端口号
    首次远程登录的时候会需要建立authenticity，确认远程主机的finger print，在信任主机的情况下同意建立authenticity，并输入密码就可以。
    新增用户并设置密码确认密码：
    >adduser username
    >passwd username
    username为新增用户的用户名，为新用户设置并确认密码后，就创建成功了

    创建用户成功后，下面需要为新用户授权，使得新用户可以用sudo语句来执行一些操作
    找到sudoers所在的位置后，修改sudoer文件的权限：
    >ls -al /etc/sudoers #查看权限
    >chmod -v u+w /etc/sudoers #修改权限以便在sudoers文件中给新用户开放权限
    下面可以在vim编辑器中为新用户开放所有权限：
    >vim /etc/sudoers
    >## Allow root to run any commands anywher
    >root    ALL=(ALL)       ALL
    >username  ALL=(ALL)       ALL  #增加这一行给新用户开放所有权限
    操作完成后，为了安全考虑，需要收回sudoers的写权限：
    >chmod -v u-w /etc/sudoers
    
    一个有所有操作权限的新用户就创建好了，下面就可以用该新用户来登录操作了

