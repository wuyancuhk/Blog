---
layout:     post
title:      "Hadoop Learning Notes - 4"
subtitle:   " \"Hadoop - How to specify python version in Linux/hadoop\""
date:       2020.11.01 23:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - Linux
    - Hadoop




---

> *"Keep Learning Hadoop"*

# How to specify python version in Linux/hadoop

## 1. Motivation

当我们在Linux或者Hadoop上运行`python`程序的时候，有时代码是用旧版本或者新版本写的，如果源代码较长，那么没法自己一行行去改，这时候就需要切换新的`python`版本。

## 2. Specify python verison in Linux

### 2.1. 安装新的python版本

这里以`python3.9`为例， `bash shell`脚本如下：

```shell
wget https://www.python.org/ftp/python/3.9.0/Python-3.9.0.tgz
tar -xzf Python-3.9.0.tgz
cd Python-3.9.0
mkdir -p /your/other/python/path   #注意这里必须明确当前用户下的路径，不然非root用户会默认装到管理员路径下，然后报错		   								  permission denied
./configure --prefix="/your/other/python/path"
make
make install
```

安装好之后可以配置下环境变量，这样每次执行时就不需要指定`python`目录了, 注意，不同的`tmux`窗口下都要重新执行一遍：

```shell
> cd ~
> vim .bashrc

# 新增下面两行
alias python3=/home/jim/software/python3.9/bin/python3.9
alias pip3=/home/jim/software/python3.9/bin/pip3.9
```

然后记得使配置文件生效：

```shell
source .bashrc
```

### 2.2. 安装pip

```shell
wget https://bootstrap.pypa.io/get-pip.py
> /home/username/.python3.9.0/bin/python get-pip.py
```

## 3. Specify python version in Hadoop

### 3.1. 将整个Python文件压缩并上传

对于上面一步下载的`python`源文件的压缩文件，上传到Hadoop个人文件系统下：

```shell
hadoop fs -put python-3.9.0.tgz hdfs:///user/your_user_id/
```

### 3.2. 明确运行Python文件的python版本

在输入MapReduce Job指令时，加入如下一行：

```shell
-archives hdfs:///user/your_user_id/python39.tar.gz
```

在明确`mapper`或者`reducer`文件的时候，以如下格式输入：

```shell
-mapper "python39.tar.gz/python39/bin/python3.9 mapper.py"
```

