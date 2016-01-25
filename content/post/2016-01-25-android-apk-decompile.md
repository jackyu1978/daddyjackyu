+++
title = "android apk反编译"
description = ""
keywords = ["",""]
categories = [""]
date = "2016-01-25T21:18:57+08:00"
url = "/2016/01/25/android-apk-decompile/"

+++

在学习Android开发的过程你，你往往会去借鉴别人的应用是怎么开发的，或者想破解或修改下别人的APK，这时，你便可以对APK进行反编译。<!--more-->

## 安装Java运行环境 ##
从Oracle官网下载JDK安装包，安装JAVA运行环境
[下载](http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html)

并配置好JAVA_HOME、PATH等环境变量。此处不多说，可以自己去google下JAVA运行环境安装及配置

## 工具准备 ##
- **[apktool](http://pan.baidu.com/s/1skuYVa1)** --资源文件获取，可以提取出图片文件和布局文件进行使用查看，反编译成smali代码
- **[dex2jar](http://pan.baidu.com/s/1jHpNCVk)** --将apk反编译成java源码（classes.dex转化成jar文件）
- **[jd-gui](http://pan.baidu.com/s/1hrq7CHY)** --查看APK中classes.dex转化成的jar文件，即Java源码文件

## 反编译步骤 ##
### 1. apk反编译得到smali程序的源代码、图片、XML配置、语言资源等文件 ###
使用apktool工具，打开命令行，进入apktool目录，输入以下命令：

    java -jar apktool.jar d -f xxx.apk

![](http://i.imgur.com/SDhL3Jo.png)
其中xxx.apk是需要反编译的apk文件。
执行成功的话在当前目录下会生成与apk同名的目录，其中包含的内容有：
smali代码目录（smali）、资源目录（res)、XML配置文件（AndroidManifest.xml）等
![](http://i.imgur.com/MAnaJ2o.png)
### 2. Apk反编译得到Java源代码 ###
将要反编译的APK后缀名改为.rar或则 .zip，并解压，得到其中的额classes.dex文件（它就是java文件编译再通过dx工具打包而成的），下载上述工具中的dex2jar，解压，在命令行下进入解压目录，输入：

    d2j-dex2jar.bat classes.dex

在该目录下会生成一个classes_dex2jar.jar的文件，然后打开工具jd-gui文件夹里的jd-gui.exe，之后用该工具打开之前生成的classes_dex2jar.jar文件，便可以看到源码了，效果如下：
![](http://i.imgur.com/jo50pue.png)

### 3.学习别人的apk代码，并尝试修改  ###
通过第二部已经看到了apk的java源代码，这时就可以认真学习下别人的apk是怎么编写的了。如果对Java代码看懂了，理解了逻辑，这时想对apk做些修改之类的，就可以到第一步反编译得出的**smali**代码目录中找到与java代码对应的smali代码，然后修改之，这时候就需要学习smali语法了。smali语法请自行google学习.

### 4.重新打包APK  ###
使用apktool工具重新打包修改后的应用（修改了smali代码后），输入：

    java -jar apktool.jar b apkdir

![](http://i.imgur.com/dv6D068.png)
### 5.重新对APK签名  ###
创建key，需要用到keytool.exe (位于jdk\jre\bin目录下)，打开cmd输入:

    keytool -genkey -alias my.keystore -keyalg RSA -validity 40000 -keystore my.keystore

**说明：**

genkey 产生密钥

alias demo.keystore 别名 demo.keystore

keyalg RSA 使用RSA算法对签名加密

validity 40000 有效期限4000天

keystore demo.keystore */

    jarsigner -verbose -keystore my.keystore -signedjar demo_signed.apk demo.apk my.keystore

**说明：**

keystore  demo.keystore 密钥库位置

signedjar demor_signed.apk demo.apk my.keystore 正式签名，三个参数中依次为签名后产生的文件demo_signed，要签名的文件demo.apk和密钥库my.keystore.

### 6.安装APK ###
修改后，重新签名的APK就可以拷贝到手机重新安装了，因为是用自己生成的key签名的，手机可能会提示病毒或木马，不用管它，安装就是。然后运行看看自己修改的功能有没有生效吧。

## 结论 ##
**注：本文只是列出了大概的步骤，反编译中重要和困难的是第3步，这个需要大量的经历和学习。**