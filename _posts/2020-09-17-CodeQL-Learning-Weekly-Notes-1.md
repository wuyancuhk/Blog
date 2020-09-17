---
layout:     post
title:      "CodeQL - Weekly Notes - 01"
subtitle:   " \"Using CodeQL to find security risks in mainstream applications\""
date:       2020.09.17 15:29:00
author:     "Wuy"
header-img: "img/post-bg-2020.jpg"
catalog: true
tags:
    - 学习笔记
    - Cryptography
    - CodeQL


---

> *"Keep Learning CodeQL"*

# CodeQL 入门

## 1. CodeQL简介

CodeQL是Github Security Lab的一个开源的代码分析引擎，主要用来分析程序漏洞，它使用`QL`语言，这是一种查询语言，支持对C++，C#，Java，JavaScript，Python，go等多种语言进行分析，可用于分析代码，查找代码中控制流等信息。

国际惯例，先简述下这工具如何使用：

首先在Visual Studio Code的插件栏找到CodeQL，下载，完成后会提示下载CodeQL Cli这一扩展，完成后在插件设置里修改下扩展的路径，就可以使用了：将在vscode-codeql-starter文件夹添加到工作区，可以看到里面有各种编程语言的查询示例，当然，前提是你要有一个用来查询的语言数据库，这个可以去到LGTM的官网上下载，这里不再赘述。

## 2. 报告阅读

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

常见的加密法里，`DES`和`3DES`是使用最多的数据块加密，`AES`是更高级的块加密法等等。

##### 2.1.3. 电码簿模式（Electronic Codebook）

`ECB`模式是分组密码的一种最基本的工作模式。在该模式下，待处理信息被分为大小合适的分组，然后分别对每一分组独立进行加密或解密处理。注意，解密手段与加密相反。

`ECB`模式作为一种基本工作模式，具有操作简单，易于实现的特点。同时由于其分组的独立性，利于实现并行处理，并且能很好地防止误差传播。

另一方面由于所有分组的加密方式一致，明文中的重复内容会在密文中有所体现，因此难以抵抗统计分析攻击。

因此，`ECB`模式一般只适用于小数据量的字符信息的安全性保护，例如密钥保护。

示例如下：

![image-20200911232818818](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200911232818.png)

##### 2.1.4. 密文分组链接模式（Cipher Block Chaining Mode）

之所以叫`CBC Mode`这个名字，是因为密文分组像链条一样相互连接在一起。在`CBC`模式中，首先将明文分组与前一个密文分组进行XOR运算，然后再进行加密。它的加解密过程如下：

![image-20200911232844446](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200911232844.png)

当加密第一个明文分组时，由于不存在“前一个密文分组”，因此需要事先准备一个长度为一个分组的比特序列来代替“前一个密文分组”，这个比特序列称为初始化向量（Initialization Vector），通常缩写为IV，一般来说，每次加密时都会随机产生一个不同的比特序列来作为初始化向量。这样一来，`ECB Mode`的缺陷在`CBC Mode`中就不存在了。

#### 2.2. 误用规则的定义

这里会介绍三个Android平台的误用规则，基于此开发的Android SDK都会轻易受到攻击.

##### 2.2.1. 误用规则一：使用`ECB`模式加密

由于`ECB`加密方式有一个致命缺陷，就是相同的明文块会加密成相同的密文。因此，它不能很好的隐藏数据模式。因为`ECB`的每一块都是使用完全相同的方式进行界面解密，这样就使得信息不能受到完全的保护，容易遭受统计分析攻击和重放攻击。另外，在Android SDK的源码中,有这样一段话(源码似乎把lookup打错成loopup了):

![image-20200909193516213](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200911232906.png)

这里如果调用`javax.crypto.Ciphere.getInstance`方法的话,如果没有明确指出加密方式,则会默认为ECB模式.此为漏洞一.

##### 2.2.2. 误用规则二：在`CBC`模式中使用固定的初始化向量

在CBC模式中,固定的初始化向量会导致`IND-CPA secure problem`,也就是不满足选择明文攻击下的不可区分性(Indistinguishability under chosen-plaintext attack), 这在解密和认证过程中是不安全的.

该性质是通过一个仿真游戏验证的，游戏过程如下:

算法拥有者称之为挑战者，算法攻击者称之为攻击者

1. 挑战者拥有公钥PK，私钥SK，将PK发给攻击者。
2. 攻击者选取长度相等的两个明文，M1，M2，发给挑战者。
3. 挑战者获得明文后，然后随机决定b的取值，$b = {0, 1}$，然后决定对于Mb的加密，$C=Enc(PK,Mb)$，然后发给攻击者。
4. 攻击者获得密文后，给出b的值，即确定是对M1的加密，还是对M2的加密。

准确得到b的值的优势$= \frac{1}{2} + \varepsilon$，如果的值可以忽略，就说明该加密算法是安全的，反之则是不安全的。

在Android SDK中, 初始化向量是被直接声明的, 声明为“13579246810123456”, 然后通过静态方法`DatatypeConverter.parseHexBinary`转化为字节数组,之后,这一数组会作为`javax.crypto.spec.IvParameterSpec`构造器方法里的一个参数被直接调用.

##### 2.2.3. 误用规则三：使用固定的密钥解密

如果密钥是固定的话, 那么这就必然可以通过暴力法破解, 这便会导致软件易受攻击.

和上一规则类似, 字符串“13579246810123456”被事先声明, 然后通过静态方法`.getBytes`或者`Base64.decode`转换成字节数组,再被作为参数传递给密钥构造器方法`javax.crypto.spec.SecretKeySpec`.

#### 2.3. 实验

根据上述三个`Code Pattern`, 我们可以基于CodeQL这一代码语义分析引擎来分析脆弱的代码.

CodeQL可以离线在VS Code上使用, 也可以在线在`LGTM`官网上使用, 这里不再赘述.

##### 2.3.1. 数据流概念（污点追踪）

数据流是这一实验中的关键概念,它包括源结点, 阱结点和结点路径. 源结点可以看成变量的声明, 阱结点可以看成一个可能存放变量的容器. 如果源结点中定义的变量可以被映射到阱结点中, 那么流迹就可以得到建立.

CodeQL提供了几种数据流的查询：

- Local data flow（DataFlow模块下面）基本是用在一个方法中的，比如想要知道一个方法的入参是否可以进入到某一个方法，就可以用它；
- Local taint data flow（TaintTracking模块下面）；
- Global data flow（继承这个类DataFlow::Configuration）是用在整个项目的，比local data flow更强大，但是也更耗时，耗内存，而且没local data flow准确，其中包含着这么几个predicates ：
  - isSource—defines where data may flow from
  - isSink—defines where data may flow to
  - isBarrier—optional, restricts the data flow
  - isAdditionalFlowStep—optional, adds additional flow steps
- Global taint data flow(继承这个类TaintTracking::Configuration)，它有这么几个predicates：
  - isSource—defines where taint may flow from
  - isSink—defines where taint may flow to
  - isSanitizer—optional, restricts the taint flow
  - isAdditionalTaintStep—optional, adds additional taint steps

##### 2.3.2.  误用规则一

利用`Misuse1.ql`对`AbrahamCaiJin_CommonUtilLibrary_51c7668`这一`Code Base`进行分析，分析代码如下：

```ql
import java
import semmle.code.java.dataflow.TaintTracking
import DataFlow


// Match the parameter called by static method or constructor
abstract class MisuseInstance extends Top { 
  MisuseInstance() { this instanceof Call}
  abstract Expr aCheck();
}

// Match the parameter called by "Cipher.getInstance"
class CipherInstance extends MisuseInstance {
  CipherInstance() {
    exists(Method method | method.getAReference() = this |
      method.getDeclaringType().getQualifiedName() = "javax.crypto.Cipher" and
      method.getName() = "getInstance"
    )
  }

  override Expr aCheck() { result = this.(MethodAccess).getArgument(0) }
}

// Match the parameter called by "KeyGenerator.getInstance" method 
class KeyGeneratorInstance extends MisuseInstance {
  KeyGeneratorInstance() {
    exists(Method method | method.getAReference() = this |
      method.getDeclaringType().getQualifiedName() = "javax.crypto.KeyGenerator" and
      method.getName() = "getInstance"
    )
  }

  override Expr aCheck() { result = this.(MethodAccess).getArgument(0) }
}

// Match the parameter called by "SecretKeySpec" constructor
class SecretKeyInstance extends MisuseInstance {
  SecretKeyInstance() {
    exists(Constructor constructor | constructor.getAReference() = this |
    constructor.getDeclaringType().getQualifiedName() = "javax.crypto.spec.SecretKeySpec" )
  }

  override Expr aCheck() {
    exists(ConstructorCall constructorCall | constructorCall = this |
    if constructorCall.getNumArgument() = 2 
    then result = constructorCall.getArgument(1)
    else result = constructorCall.getArgument(3)
    )
  }
}

// Match the parameter that could lead to unsafety
abstract class MisusePara extends StringLiteral {
  MisusePara() { getLiteral().length() != 0}
}

// The first term of encryption parameter
string symmetricEncryption_first() {
  result = "AES" or
  result = "DES" or
  result = "DESede" or
  result = "HmacSHA1" or
  result = "HmacSHA256"
}

// The second term of encryption parameter
string symmetricEncryption_second() {
  result = "ECB"
}

// Search first parameter based on index
string first_para(int i) {
  result = rank[i](symmetricEncryption_first())
}

// Search second parameter based on index
string second_para(int i) {
  result = rank[i](symmetricEncryption_second())
}

// Combine the parameters to conduct regexmatch
string combine_paras(int i, int j) {
  if j = 0
  then result = first_para(i) 
  else result = first_para(i) + "/" + second_para(j)
}

// Match the parameter that specify the "ECB" mode
class ECB_Encryption extends MisusePara{
    ECB_Encryption() {
      getValue().regexpMatch("(^|.*[^A-Z])(" + combine_paras(1, 1) + ")([^A-Z].*|$)") or 
      getValue().regexpMatch("(^|.*[^A-Z])(" + combine_paras(2, 1) + ")([^A-Z].*|$)") or
      getValue().regexpMatch("(^|.*[^A-Z])(" + combine_paras(3, 1) + ")([^A-Z].*|$)") or
      getValue().regexpMatch("(^|.*[^A-Z])(" + combine_paras(4, 1) + ")([^A-Z].*|$)") or
      getValue().regexpMatch("(^|.*[^A-Z])(" + combine_paras(5, 1) + ")([^A-Z].*|$)") 
    }
  }

// Match the parameter of "AES" encryption with default "ECB" mode
class Default_Encryption extends MisusePara{
  Default_Encryption() {
    getValue().regexpMatch("^|" + symmetricEncryption_first() + "|$")
    //getValue().regexpMatch("^|" + combine_paras(2, 0) + "|$")
  }
}

// DataFlow: source to match the parameter and sink to match the parameter call 
class InsecureCryptoConfiguration extends TaintTracking::Configuration {
  InsecureCryptoConfiguration() { this = "BrokenCryptoAlgortihm::InsecureCryptoConfiguration" }

  override predicate isSource(Node n) { n.asExpr() instanceof MisusePara }

  override predicate isSink(Node n) { exists(MisuseInstance misuseInstance | n.asExpr() = misuseInstance.aCheck()) }

  override predicate isSanitizer(DataFlow::Node node) {
    node.getType() instanceof PrimitiveType or node.getType() instanceof BoxedType
  }
}


from
  PathNode source, 
  PathNode sink, 
  MisuseInstance misuseInstance, 
  MisusePara misusePara,
  // ECB_Encryption ecb , 
  // Default_AES_Encryption aes,
  InsecureCryptoConfiguration conf

where
  source.getNode().asExpr() = misusePara and
  sink.getNode().asExpr() = misuseInstance.aCheck() and
  conf.hasFlowPath(source, sink)
  //misuse.aCheck() = ecb
  //misuse.aCheck() = aes

//select misuse, ecb
 select source, sink

```

运行后找出了共计19个符合条件的结果，都是包含了`ECB`加密模式或者会默认填充该模式的加密算法代码：

![image-20200917161714679](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917161721.png)

可以直接通过高亮的结果定位到问题代码的位置，下面是其中一段问题代码：

```java
/*
     * DES 加密
     */
    public static byte[] encrypt(byte[] data, byte[] key) throws Exception {
        SecretKey secretKey = new SecretKeySpec(key, "DES");

        Cipher cipher = Cipher.getInstance("DES/ECB/PKCS5Padding");
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        byte[] cipherBytes = cipher.doFinal(data);
        return cipherBytes;
    }
```

##### 2.3.3. 误用规则二

利用`Misuse2.ql`对`structurizr_java_java-src`这一`Code Base`进行分析，发现`AesEncryptionStrategy.java`文件里存在两处误用：

![image-20200912000320438](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200912000320.png)

根据显示的结果，两个声明为`iv`的字符串变量经过`parseHexBinary()`方法传递到构造器方法`IvParameterSpec()`.

`Misuse2.ql`的代码具体如下：

```ql
import java
import semmle.code.java.dataflow.DataFlow

abstract class MisusePara extends Top { 
  MisusePara() { this instanceof Call}
  abstract Expr pCheck();
}
class IvParameterInstance extends MisusePara {
  IvParameterInstance() {
    exists(Method method | method.getAReference() = this |
      method.getDeclaringType().getQualifiedName() = "javax.xml.bind.DatatypeConverter" and
      method.getName() = "parseHexBinary"
    )
  }

  override Expr pCheck() { result = this.(MethodAccess).getArgument(0) }
}



from Constructor c, Variable source, Call sink, MisusePara mp
where c.getDeclaringType().hasQualifiedName("javax.crypto.spec", "IvParameterSpec")
and sink.getCallee() = c
// ma.getMethod() = m
// and m.getDeclaringType().hasQualifiedName("javax.xml.bind.DatatypeConverter", "parseHexBinary")
// and DataFlow::localFlow(DataFlow::exprNode(mp), DataFlow::exprNode(sink.getArgument(0))) //and src_iv.getType().hasName("byte[]")
and DataFlow::localFlow(DataFlow::exprNode(sink.getArgument(0)), DataFlow::exprNode(mp))
and DataFlow::localFlow(DataFlow::exprNode(mp.pCheck()), DataFlow::exprNode(source.getAnAccess())) 
and source.getType() instanceof TypeString
select source, sink
```

文件中第一个问题代码如下所示，可以看到， 初始化向量`iv`会被事先声明为一个固定值：

```java
	public AesEncryptionStrategy(int keySize, int iterationCount, String salt, String iv, String passphrase) 
	{
        super(passphrase);

        this.keySize = keySize;
        this.iterationCount = iterationCount;
        this.salt = salt;
        this.iv = iv;
    }
```

进而该值会被传递到加密方法`encrypt()`中：

```java
public String encrypt(String plaintext) throws Exception {
        SecretKey secretKey = createSecretKey();

        Cipher cipher = Cipher.getInstance(CIPHER_SPECIFICATION);
        cipher.init(Cipher.ENCRYPT_MODE, secretKey, new IvParameterSpec(DatatypeConverter.parseHexBinary(iv)));//这里用到了上面提到的初始化向量iv

        byte[] byteDataToEncrypt = plaintext.getBytes();
        byte[] byteCipherText = cipher.doFinal(byteDataToEncrypt);

        return Base64.getEncoder().encodeToString(byteCipherText);
    }
```



##### 2.3.4. 误用规则三

利用`Misuse3.ql`对`aws-amplify_aws-sdk-android_a131321`这一`Code Base`进行分析，发现共三处误用：

![image-20200917174514384](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917174514.png)

`Misuse3.ql`文件代码如下：

```
import java
import semmle.code.java.dataflow.DataFlow

abstract class Misuse_3 extends Top { 
  Misuse_3() { this instanceof Call}
  abstract Expr pCheck();
}

class Case_1 extends Misuse_3 {
  Case_1() {
    exists(Method method | method.getAReference() = this |
      
      method.getName() = "getBytes"
    )
  }

  override Expr pCheck() { result = this.(MethodAccess).getQualifier() }
}

class Case_2 extends Misuse_3 {
  Case_2() {
    exists(Method method | method.getAReference() = this |
      method.getDeclaringType().getQualifiedName() = "com.amazonaws.util.Base64" and
      method.getName() = "decode"
    )
  }

  override Expr pCheck() { result = this.(MethodAccess).getArgument(0) }
}


from Constructor c, Variable source, Call sink, Misuse_3 method
where c.getDeclaringType().hasQualifiedName("javax.crypto.spec", "SecretKeySpec")
and sink.getCallee() = c
// ma.getMethod() = m
// and m.getDeclaringType().hasQualifiedName("javax.xml.bind.DatatypeConverter", "parseHexBinary")
and DataFlow::localFlow(DataFlow::exprNode(method), DataFlow::exprNode(sink.getArgument(0))) //and src_iv.getType().hasName("byte[]")
and DataFlow::localFlow(DataFlow::exprNode(source.getAnAccess()), DataFlow::exprNode(method.pCheck())) 
and source.getAnAccess().getType() instanceof TypeString
select source, method, sink
```

对于找出的两个符合误用规则的方法代码，它们的具体数据流向如下所示：

对于第一个不安全的方法`getBytes()`，在静态方法`getSecretHash`中会声明变量`clientSecret`，而这一字段会被事先声明为一个固定值：

```java
public static String getSecretHash(String userId, String clientId, String clientSecret)
{
    ...
}
```

进而该值会通过`getBytes()`方法的转换被传递到密钥构造方法`SecretKeySpec()`中：

```java
final SecretKeySpec signingKey = new SecretKeySpec(clientSecret.getBytes(StringUtils.UTF8),
                HMAC_SHA_256);
```

同样的，对于`Base64.decode()`方法，它的传播路径如下：

首先， 在变量`keyInStringFormat`的构造方法中会调用固定初始变量`keyAlias`：

```java
final String keyInStringFormat = sharedPreferences.getString(keyAlias, null);
```

然后改变量会通过`Base64.decode()`方法生成一个字节数组`base64DecodedAESEncryptionKey`：

```java
byte[] base64DecodedAESEncryptionKey = Base64.decode(keyInStringFormat);
```

最后，该字节数组在密钥生成方法`retrieveKey()`中调用上面中提到的方法`SecretKeySpec()`，而该方法的参数就是刚刚生成的字节数组`base64DecodedAESEncryptionKey`：

```java
public synchronized Key retrieveKey(final String keyAlias) throws KeyNotFoundException 
{
	...
	if (base64DecodedAESEncryptionKey == null || base64DecodedAESEncryptionKey.length == 0) 
    {
        throw new KeyNotFoundException("Error in Base64 decoding the AES encryption key " +
                                       "identified by the keyAlias: " + keyAlias);
    }

    	return new SecretKeySpec(base64DecodedAESEncryptionKey, AES_KEY_ALGORITHM); //这里调用了问题数组
	...
}
```

## 3. CodeQL Demo for Python

我自己编写了一个Demo来查询`exercism_python_42c2c49`（LGTM上的代码数据库）这一`Code Base`里出现的重复的键：

```
import python
 
predicate founder_same_keys(Key k1, Key k2) 
{
  k1.(Num).getN() = k2.(Num).getN()
  or
  k1.(StrConst).getText() = k2.(StrConst).getText()
}
 
from Dict d, Expr k1, Expr k2
where k1 = d.getAKey() 
and k2 = d.getAKey()
and k1 != k2 
and founder_same_keys(k1, k2)
select k1, "Duplicate key in the codebase"
```

运行结果如下：

![image-20200917212420269](https://raw.githubusercontent.com/Lov3Camille/postimage/master/20200917212420.png)

