---
title: 搭建自己的linux工作环境
date: 2019-09-30 18:07:02
categories:
- 工具
tags:
- linux
- 工作环境

---

## 缘由
由于进公司之后长期使用win10操作系统，在win10操作系统不停强制更新之后，终于在2019年8月电脑开始频繁蓝屏，几乎每天一次！
痛定思痛，决定放弃win10系统，采用win7作为主要操作系统，使用虚机的方式多系统联合作战。



intel wifi linux驱动包
https://www.intel.com/content/www/us/en/support/articles/000005511/network-and-io/wireless-networking.html

升级内核
1.自行下载内核，然后采用yum localinstall进行安装
4.15内核
http://mirror.host.ag/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-4.15.14-1.el7.elrepo.x86_64.rpm

2.联网后配置yum源进行安装
导入key
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
安装yum源
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
查看系统可以使用的内核
sudo awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg


删除不必要的内核
yum remove kernel-3.10.0-327.el7.x86_64


office常用办公软件
yum install libreoffice

centos7.1没有wifi-GUI GUI联网得用wap_suplication

切换到centos7.6

安装vmware player



.......未完待续