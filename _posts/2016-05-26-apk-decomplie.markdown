---
layout:     post
title:      "Android反编译总结"
subtitle:   " \"Android反编译总结\""
date:       2016-05-26 12:00:00
author:     "X-ray"
header-img: "img/post-bg-2015.jpg"
tags:
    - Android
---



> 工欲善其事必先利其器

## 前言

Android反编译需要用到一些工具，使用这些工具可以大大的提高我们的工作效率。当然使用工具是一把双刃剑，一方面工具使我们的工作更方便，但另一方面工具把一些原理封装了起来，不利于我们的学习。所以使用工具的时候最好对工具的原理有一定的了解。



## jadx

[github地址](https://github.com/skylot/jadx)，jadx是一个将android 的dex文件解码为java的工具。并且有GUI，用起来非常的方便,具体的使用可以看官方的文档。

## apktool

有了上面的工具，可以很方便的将apk的源码进行反编译，让我们了解代码的逻辑。但是这还远远不够，比如我们想干点坏事，将apk反编译后，做点手脚，并把它重新打包发布出去，要实现这样的功能，光上面的工具已经达不到我们的要求了。我们需要另外一个更加强大的工具－[apktool](http://ibotpeaches.github.io/Apktool/install/).  apktool可以将dex文件反编译成smali文件。如果对smali语法熟悉，就可以在smali中修改app的逻辑。其实apktool这个工具集成了另外一个开源的项目[baksmali](https://github.com/JesusFreke/smali)，(因为apktool集成了baksmail， 假如baksmail修复了bug，需要一段时间才能集成的apktool中，所以我们也可以直接使用baksmail)



## smali语法

为了达到我们对一个apk做手脚的目的，需要了解一种新的语言smali, smail语言类似与汇编语言，如果对汇编比较熟悉的学习起来会比较容易。



## 实战

下面就举个例子，演示一下怎么利用apktool解包，修改smali文件，重新打包，签名的过程。

首先新建个项目，主要一个MainActivity，里面只有一个字符串，我的目的就是通过解包apk,在smali语法中注入日志打印语句，把这个字符串打印出来:



```

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        String hello = "Hello World";

    }
}
```



将项目打包出来的apk（debug包和release包都可以）拷贝到apktool的同级目录



![目录](http://7xrrm5.com1.z0.glb.clouddn.com/blog_pic_article_decompile_apk_1.png)



然后将apk包用apktool，反编译，命令为：



```

cfp@cfp:~/tools/android_tools/apktool$ ./apktool d app-release.apk 

I: Using Apktool 2.1.1 on app-release.apk

I: Loading resource table...

I: Decoding AndroidManifest.xml with resources...

I: Loading resource table from file: /home/cfp/apktool/framework/1.apk

I: Regular manifest package...

I: Decoding file-resources...

I: Decoding values */* XMLs...

I: Baksmaling classes.dex...

I: Copying assets and libs...

I: Copying unknown files...

I: Copying original files...

cfp@cfp:~/tools/android_tools/apktool$ 

```

反编译后多了个app-release目录，反编译的文件就都在里边了，下面的任务就是找到MainActivity，在里边注入日志打印语言，将里边的字符串输出。



```

cfp@cfp:~/tools/android_tools/apktool$ ls

apktool      app-release      debug.keystore  sign.jar

apktool.jar  app-release.apk  signapk.jar

```



反编译后的目录：



```

cfp@cfp:~/tools/android_tools/apktool/app-release$ ls

AndroidManifest.xml  apktool.yml  original  res  smali

```



所有的smali文件都在smali目录下，我们找到MainActivity.smali文件



```

cfp@cfp:~/tools/android_tools/apktool/app-release/smali/com/xray/smali$ ls

BuildConfig.smali   R$bool.smali      R$id.smali       R.smali

MainActivity.smali  R$color.smali     R$integer.smali  R$string.smali

R$anim.smali        R$dimen.smali     R$layout.smali   R$styleable.smali

R$attr.smali        R$drawable.smali  R$mipmap.smali   R$style.smali

```



MainActivity.smali文件中的代码：



```

.class public Lcom/xray/smali/MainActivity;　＃class的名字

.super Landroid/support/v7/app/AppCompatActivity;　＃这个类的父类

.source "MainActivity.java"　＃这个类的java文件名





# direct methods

.method public constructor <init>()V　＃这个类的构造方法

    .locals 0



    .prologue

    .line 7　＃行号

    invoke-direct {p0}, Landroid/support/v7/app/AppCompatActivity;-><init>()V　＃调用父类的构造方法



    return-void　＃返回空

.end method





# virtual methods

.method protected onCreate(Landroid/os/Bundle;)V　＃onCreate方法

    .locals 2

    .param p1, "savedInstanceState"    # Landroid/os/Bundle;　#方法的参数



    .prologue

    .line 11

    invoke-super {p0, p1}, Landroid/support/v7/app/AppCompatActivity;->onCreate(Landroid/os/Bundle;)V　

＃父类的构造方法





    .line 12

    const v1, 0x7f040019



    invoke-virtual {p0, v1}, Lcom/xray/smali/MainActivity;->setContentView(I)V　#调用serContentView方法



    .line 13

    const-string v0, "Hello World"　＃字符串Hello World，保存在寄存器v0中



    .line 15

    .local v0, "hello":Ljava/lang/String;　＃变量hello ,类型为String

    return-void

.end method





```

前面说了smali语言有点像汇编，通过上面的注释大概能了解这个smali文件的大概意思，要想输出Hello World，我么可以加上调用Log的smali语句。



```

const-string v1, "TAG"

invoke-static {v1, v0}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I

```

修改后的MainActivity.smali文件变成：



```

.class public Lcom/xray/smali/MainActivity;　＃class的名字

.super Landroid/support/v7/app/AppCompatActivity;　＃这个类的父类

.source "MainActivity.java"　＃这个类的java文件名





# direct methods

.method public constructor <init>()V　＃这个类的构造方法

    .locals 0



    .prologue

    .line 7　＃行号

    invoke-direct {p0}, Landroid/support/v7/app/AppCompatActivity;-><init>()V　＃调用父类的构造方法



    return-void　＃返回空

.end method





# virtual methods

.method protected onCreate(Landroid/os/Bundle;)V　＃onCreate方法

    .locals 2

    .param p1, "savedInstanceState"    # Landroid/os/Bundle;　#方法的参数



    .prologue

    .line 11

    invoke-super {p0, p1}, Landroid/support/v7/app/AppCompatActivity;->onCreate(Landroid/os/Bundle;)V　

＃父类的构造方法





    .line 12

    const v1, 0x7f040019



    invoke-virtual {p0, v1}, Lcom/xray/smali/MainActivity;->setContentView(I)V　#调用serContentView方法



    .line 13

    const-string v0, "Hello World"　＃字符串Hello World，保存在寄存器v0中



    .line 15

    .local v0, "hello":Ljava/lang/String;　＃变量hello ,类型为String

    const-string v1, "TAG"

    invoke-static {v1, v0}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I

    return-void

.end method

```



然后重新打包:



```

cfp@cfp:~/tools/android_tools/apktool$ ./apktool b app-release

I: Using Apktool 2.1.1

I: Checking whether sources has changed...

I: Smaling smali folder into classes.dex...

I: Checking whether resources has changed...

I: Building resources...

I: Building apk file...

I: Copying unknown files/dir...

```

新的apk包位于



```

app-release/dist

```

这个apk包是不能直接安装的，需要用签名工具签名，可以使用另外一个工具[sign.jar](https://github.com/appium/sign),命令为：



```

cfp@cfp:~/tools/android_tools/apktool$ java -jar sign.jar app-release.apk 



```

签名后的新包叫app-release.s.apk，直接安装就可以打印Hello World的日志了。

﻿

```

05-25 22:56:25.725 17277-17277/com.xray.smali D/TAG: Hello World

```









































## 参考文献

1. [smali语法](http://blog.isming.me/2015/01/14/android-decompile-smali/)

2. http://drops.wooyun.org/papers/6045




































































