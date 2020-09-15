---
layout:     post
title:      "CodeQL - Daily Notes - 01"
subtitle:   " \"Using CodeQL to find security risks in mainstream applications\""
date:       2020.09.09 15:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - 论文阅读
    - Cryptography


---

> *"Keep Learning CodeQL"*

# CodeQL 入门

## 1. CodeQL简介

CodeQL是Github Security Lab的一个开源的代码分析引擎，主要用来分析程序漏洞，它使用`QL`语言，这是一种查询语言，支持对C++，C#，Java，JavaScript，Python，go等多种语言进行分析，可用于分析代码，查找代码中控制流等信息。

国际惯例，先简述下这工具如何使用：

首先在Visual Studio Code的插件栏找到CodeQL，下载，完成后会提示下载CodeQL Cli这一扩展，完成后在插件设置里修改下扩展的路径，就可以使用了：将在vscode-codeql-starter文件夹添加到工作区，可以看到里面有各种编程语言的查询示例，当然，前提是你要有一个用来查询的语言数据库，这个可以去到LGTM的官网上下载，这里不再赘述。

## 2. 论文阅读

关于这个工具的使用背景，我们需要通过一些前任的研究成果来深入了解，然后再开始我们自己的工作。首先是上一届学长Fang Ming的研究报告。

### 《基于CodeQL的不安全加密检测》

#### 2.1. 简介

这篇研究报告主要是研究Android应用中一些不安全的加密方法。

##### 2.1.1. 对称加密（Symmetric Encryption）

对称加密(也叫私钥加密)指加密和解密使用相同密钥的加密算法。有时又叫传统密码算法，就是加密密钥能够从解密密钥中推算出来，同时解密密钥也可以从加密密钥中推算出来。而在大多数的对称算法中，加密密钥和解密密钥是相同的，所以也称这种加密算法为秘密密钥算法或单密钥算法。它要求发送方和接收方在安全通信之前，商定一个密钥。对称算法的安全性依赖于密钥，泄漏密钥就意味着任何人都可以对他们发送或接收的消息解密，所以密钥的保密性对通信的安全性至关重要。

如下如所示：

![image-20200911233006210](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200911233006.png)

##### 2.1.2. 块加密（Block Cipher）

块加密，又称分组加密，是一种常见的对称加密。它将固定长度的数据块或纯文本数据（未加密）转换成长度相同的密码块（加密文本）数据。该转换的前提是用户提供密钥。解密时，要使用相同的密钥对密码块数据进行逆转换。固定的长度被称做数据块大小，大多数密码块的固定大小都是64位或128位。

块加密的工作模式允许使用同一个分组密码密钥对多于一块的数据进行加密，并保证其安全性，一般来说块加密相比较刘加密更安全一些。

常见的加密法里，DES和3DES是使用最多的数据块加密，AES是更高级的块加密法等等。

##### 2.1.3. 电码簿模式（Electronic Codebook）

ECB模式是分组密码的一种最基本的工作模式。在该模式下，待处理信息被分为大小合适的分组，然后分别对每一分组独立进行加密或解密处理。注意，解密手段与加密相反。

ECB模式作为一种基本工作模式，具有操作简单，易于实现的特点。同时由于其分组的独立性，利于实现并行处理，并且能很好地防止误差传播。

另一方面由于所有分组的加密方式一致，明文中的重复内容会在密文中有所体现，因此难以抵抗统计分析攻击。

因此，ECB模式一般只适用于小数据量的字符信息的安全性保护，例如密钥保护。

示例如下：

![image-20200911232818818](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200911232818.png)

##### 2.1.4. 密文分组链接模式（Cipher Block Chaining Mode）

之所以叫CBC Mode这个名字，是因为密文分组像链条一样相互连接在一起。在CBC模式中，首先将明文分组与前一个密文分组进行XOR运算，然后再进行加密。它的加解密过程如下：

![image-20200911232844446](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200911232844.png)

当加密第一个明文分组时，由于不存在“前一个密文分组”，因此需要事先准备一个长度为一个分组的比特序列来代替“前一个密文分组”，这个比特序列称为初始化向量（Initialization Vector），通常缩写为IV，一般来说，每次加密时都会随机产生一个不同的比特序列来作为初始化向量。这样一来，ECB Mode的缺陷在CBC Mode中就不存在了。

#### 2.2. 误用规则的定义

这里会介绍三个Android平台的误用规则，基于此开发的Android SDK都会轻易受到攻击.

##### 2.2.1. 规则一：使用ECB模式加密

由于密文的不可区分性,不应该使用这种不安全的加密方式.在Android SDK的源码中,有这样一段话(源码似乎把lookup打错成loopup了):

![image-20200909193516213](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200911232906.png)

这里如果调用`javax.crypto.Ciphere.getInstance`方法的话,如果没有明确指出加密方式,则会默认为ECB模式.此为漏洞一.

##### 2.2.2. 规则二：在CBC模式中使用固定的初始化向量

在CBC模式中,固定的初始化向量会导致IND-CPA secure problem,也就是不满足选择明文攻击下的不可区分性(Indistinguishability under chosen-plaintext attack), 这在解密和认证过程中是不安全的.

该性质是通过一个仿真游戏验证的，游戏过程如下:

算法拥有者称之为挑战者，算法攻击者称之为攻击者

1. 挑战者拥有公钥PK，私钥SK，将PK发给攻击者。
2. 攻击者选取长度相等的两个明文，M1，M2，发给挑战者。
3. 挑战者获得明文后，然后随机决定b的取值，$b = {0, 1}$，然后决定对于Mb的加密，$C=Enc(PK,Mb)$，然后发给攻击者。
4. 攻击者获得密文后，给出b的值，即确定是对M1的加密，还是对M2的加密。

准确得到b的值的优势$= \frac{1}{2} + \varepsilon$，如果的值可以忽略，就说明该加密算法是安全的，反之则是不安全的。

在Android SDK中, 初始化向量是被直接声明的, 声明为“13579246810123456”, 然后通过静态方法`DatatypeConverter.parseHexBinary`转化为字节数组,之后,这一数组会作为`javax.crypto.spec.IvParameterSpec`构造器方法里的一个参数被直接调用.

##### 2.2.3. 规则三：使用固定的密钥解密

如果密钥是固定的话, 那么这就必然可以通过暴力法破解, 这便会导致软件易受攻击.

和上一规则类似, 字符串“13579246810123456”被事先声明, 然后通过静态方法`.getBytes`或者`Base64.decode`转换成字节数组,再被作为参数传递给构造器方法`javax.crypto.spec.SecretKeySpec`.

#### 2.3. 实验

根据上述三个`Code Pattern`, 我们可以基于CodeQL这一代码语义分析引擎来分析脆弱的代码.

CodeQL可以离线在VS Code上使用, 也可以在线在`LGTM`官网上使用, 这里不再赘述.

##### 2.3.1. 数据流

数据流是这一实验中的关键概念,它包括源结点, 阱结点和结点路径. 源结点可以看成变量的声明, 阱结点可以看成一个可能存放变量的容器. 如果源结点中定义的变量可以被映射到阱结点中, 那么流迹就可以得到建立.

##### 2.3.2.  误用规则一

##### 2.3.3. 误用规则二

利用`Misuse2.ql`对`structurizr_java_java-src`这一`CodeBase`进行分析，发现`AesEncryptionStrategy.java`文件里存在两处误用：

![image-20200912000320438](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200912000320.png)

根据显示的结果，两个声明为`iv`的字符串变量经过`parseHexBinary()`方法传递到构造器方法`IvParameterSpec()`.

文件中误用代码的位置如下图：

![image-20200912000902498](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200912000902.png)

##### 2.3.4. 误用规则三
