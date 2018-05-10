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
    >Changing password for user zln.
    >New password:
    >Retype new password:
    >passwd: all authentication tokens updated successfully.

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

###二.配置ssh密钥对登录

    首先需要在本地的机器上生成一对密钥对，可以在目录下/home/centos/.ssh/下检查有没有已经生成的id_rsa文件和id_rsa.pub公私钥，有的话可以直接使用，也可以重新生成一对，没有或者重新生成的话，在该目录下运行如下语句即可

    >ssh-keygen -t rsa

    或者详细的密钥对生成的方法可以参考[github上的help文档](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

    密钥对生成以后，需要将公钥，也就是id_rsa.pub文件放到远程服务器的/home/zln/.ssh下

    >scp -P 端口号 -r /Users/本地用户名/.ssh/id_rsa.pub 远程主机用户名@远程主机IP地址:/home/远程主机用户名/.ssh

    接着用新创建的用户名登录远程主机，将公钥的文件名改成authorized_keys

    >mv id_rsa.pub authorized_keys

    下面修改公钥文件的权限和这个目录的权限

    >chmod 600 ./authorized_keys
    >chmod 700 /home/username/.ssh

    接着配置/etc/ssh/sshd_config目录下的配置文件，去掉下面三行前面的#号：

    >#RSAAuthentication yes
    >#PubkeyAuthentication yes
    >#AuthorizedKeysFile .ssh/authorized_keys

    修改好以后，需要重启ssh服务来使ssh登录生效

    >systemctl restart sshd.service

    现在就可以试一下是否能够不使用密码登录了
    如果谨慎一点，可以在确认ssh登录能够生效之后，将密码登录关闭，防止黑客攻破密码后使用密码登录，具体操作方法很简单：
    将/etc/ssh/sshd_config中的控制密码登录的部分进行修改

    >PasswordAuthentication yes

    把上面的yes改成no就可以

    还有要注意的一点是，如果用新的terminal窗口进行ssh登录，要重新add一下本地私钥，在本地输入以下两条命令：

    >eval "$(ssh-agent -s)"
    >ssh-add -K ~/.ssh/id_rsa



