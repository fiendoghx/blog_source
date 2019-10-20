---
title: 大文本文件处理-Tar命令压缩_解压缩
date: 2019-10-13 11:37:34
categories:
- 工具
tags:
- Linux
- Tar
- 压缩解压缩
- 加密解密
- 大文本
---


### 目录
* [1.缘由](#缘由)
* [2.目的](#目的)
* [3.具体实施](#目的)
    * [3.1 准备工具](#准备工具)
    * [3.2 实施并验证](#实施并验证)
* [4.为什么采用以上方式](#为什么采用以上方式)

## 缘由

近期接到一个需求，需要从内网提取数据，数据是文本格式的，由于内网速度受限，
首先想到的就是压缩文件，于是研究了一下压缩文本传输的方式。

## 目的

在Linux中对大本文文件进行压缩，并加密解密

## 具体实施

## 准备工具

* 系统：centos7
* 工具包: tar,openssl
* 待处理的文件: test.txt (本文准备的文件大小在4G左右)

## 实施并验证

tar指令核心参数：

|参数  | 说明 |
| :----------:| :-----------:| 
|-j   |以bzip2方式压缩      |
|-z   |以gzip方式压缩      |
|-c   |建立新的备份文件      |
|-f   |指定备份文件      |
|-v   |显示指令执行过程      |
|-x   |从备份文件中还原文件      |



**1.使用tar进行压缩和解压**

```
压缩： tar -jcvf ./test.tar.bz2 ./test.txt

解压： tar -jxf ./test.tar.bz2
```

**2.使用tar + openssl进行压缩和解压**

```
压缩： tar -jcvf ./test.txt | openssl des3 -salt -k <password> -out ./test.tar.bz2

解压： openssl des3 -d -k <password> -salt -in /test.tar.bz2 | tar jxf -
```

## 为什么采用以上方式

**采用bzip2**

相对于gzip压缩速度慢，但压缩效果好

**采用gzip**

相对bzip2压缩速度快，但压缩效果差

**对比实例**

压缩981M的text.txt

**gzip：**

time tar  -zcvf ./test.tar.gz ./test.txt

real    0m22.177s
user    0m21.874s
sys     0m1.006s

131M test.tar.gz

**bzip2：**

time tar -jcvf test.tar.bz2 ./test.txt

real    3m1.416s
user    3m0.875s
sys     0m1.638s

99M test.tar.bz2