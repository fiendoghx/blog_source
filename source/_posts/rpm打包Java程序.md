---
title: rpm打包Java程序
categories:
  - Other
tags:
  - Other
date: 2021-04-01 15:05:31
---

### 目录
* [1.目的](#1-目的)
* [2.具体实施](#2-具体实施)
    * [2.1 事前准备](#2-1事前准备)
    * [2.2 实施并验证](#2-2实施并验证)
* [3.为什么采用以上方式](#3-为什么采用以上方式)



## 1.目的

> 没戏，但就是干！

## 2.具体实施


### 2.1事前准备

0.了解rpm打包相关知识，推荐先看官方再看其他资料 http://ftp.rpm.org/max-rpm/p5206.html

1.准备vmware centos环境，装好rpm-build

可以使用centos环境直接 yum install rpmdevtools -y

yum源设置移步 [传送门](https://)

rpmdevtools 包含了rpm-build 以及其他的环境

若是内网环境，推荐在外网建立虚拟机安装好后打包发到内网运行

2.准备打包成jar的java程序

### 2.2实施并验证

1.建立build相关目录

```
安装rpmdevtools之后，使用rpmdev-setuptree
或mkdir -p ~/rpmbuild/{BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS}
```

2.放入源程序

```
将java程序打包为tar包仍到SOURCE目录中(开始打包后会rpm会解压源文件放入BUILD中以待使用)
```

3.编写SPEC

```
vim ~/SPECS/test.spec
vim会自动创建一个有模板的spec文件，具体写法参考官方文档，此处只贴大致运行的文件

例子：


Summary: Amanda Network Backup System
Name: amanda
Version: 2.3.08
Release: 1

Group: System/Backup
License: BSD-like, but see COPYRIGHT file for details
Packager: Edward C. Bailey <bailey@rpm.org>
URL: http://www.cs.umd.edu/projects/amanda/
# 可以填写自己的SOURCE文件
Source: ftp://ftp.cs.umd.edu/pub/amanda/amanda-2.3.0.tar.gz 

%description
Amanda is a client/server backup system.  It uses standard tape
devices and networking, so all you need is any working tape drive
and a network.  You can use it for local backups as well.

%define today $(date +%y%m%d%H%M)

# 避免解压源文件时会按照版本号去访问目录，配合%setup -n amanda 可以忽略后面版本来创建目录
%define debug_package %{nil}
%define __jar_repack 0

%prep
%setup -n amanda

%build
# 由于是java程序，不需要编译，此处不做处理

%install
#删除之前异常残留的文件，并把处理后的源文件发到BUILDROOT中，BUILDROOT作为虚拟目录，就是以后rpm包中的内容
#rpmbuild异常时，会把文件保留到BUILDROOT中，需要删除后再进行cp
rm -rf %{buildroot}
mkdir -p %{buildroot}
cp -rf * %{buildroot}


#此处可以对文件进行其他的处理，例如脚本权限chmod，或将源文件中bin注册到service等
# cp -f %{buildroot}/systemctl/*.service %{buildroot}/usr/lib/systemd/system/
# cp -f %{buildroot}/bin/run %{buildroot}/etc/init.d/


%files
%doc
# 此处需要列举所有的文件，rpmbuild最后会按照此处去验证rpm包是否有此文件，最后安装时也会按照此目录安装各个文件
/README.md
/xxx.jar
/bin/run
/systemctl/run.service

```

4.执行打包获取rpm

```
cd ~/SPECS/
rpmbuild -bb test.spec

此处可能由于检测到程序中使用了相对路径rpath 而报ERROR 0002
可以使用 vi ~/.rpmmacros
注释掉%__arch_install_post 这行即可

打包后在~/rpmbuild/RPMS中可以得到自己的rpm包
```


5.执行rpm安装

```
rpm -ivh xxx.rpm
```

6.执行具体java程序以验证安装成功

```
java -jar xxx.jar
```

## 3.为什么采用以上方式

没啥，简化安装流程

