---
layout: post
title:      "[集]android.mk分析"
subtitle:   "NDK编译动态库 "
tags:       [Android ,NDK]
mathjax:    false
date:       2017-2-19
author:     "Undefined"
description: ""
header-img: "/img/tags-bg.jpg"
categories:  [Android  ]
---
## 目录
{: .no_toc}

* 目录
{:toc}


> 本篇日记皆为网上转载复制..参考网址见附录参考

---

在Android NDK开发中，有两个重要的文件：`Android.mk和Application.mk`，各尽其责，指导编译器如何编译程序，并决定编译结果是什么。


Android.mk中包括`NDK`提供的`宏`、`变量`以及模块描述变量，这些宏、变量以及变量的赋值共同组成了Android.mk文件，其在NDK编译中各尽其责，指导着NDK的编译。
宏：包括my-dir、all-subdir-makefiles等，通过‘$(call <function>)’来调用，返回文本信息。


变量：包括CLEAR_VARS、BUILD_SHARED_LIBRARY、TARGET_ARCH等，由NDK编译系统提供，并且在Android.mk文件被解析前就已经存在。Android.mk文件有可能被多次解析，因此每次解析时这些变量的值都有可能不同。
模块描述变量：Module-description，包括LOCAL_PATH、LOCAL_MODULE、LOCAL_SRC_FILES等LOCAL_前缀变量，
这些变量除LOCAL_PATH外，均填写在语句`include $(CLEAR_VARS)和include $(BUILD_XXX)`之间。


### 1.Android.mk 一个例子

	LOCAL_PATH := $(call my-dir) 
	include $(CLEAR_VARS)
	LOCAL_MODULE    := SZZTPaymentSRC
	LOCAL_C_INCLUDES += $(LOCAL_PATH)
	LOCAL_C_INCLUDES += $(LOCAL_PATH)/hsm/crypt
	LOCAL_C_INCLUDES += $(LOCAL_PATH)/hsm/dukpt_cli
	LOCAL_C_INCLUDES += $(LOCAL_PATH)/hsm/dukpt_common
	LOCAL_C_INCLUDES += $(LOCAL_PATH)/hsm/pkcrypt_cli
	ALL_FILES := $(wildcard $(LOCAL_PATH)/*.c $(LOCAL_PATH)/hsm/crypt/*.c $(LOCAL_PATH)/hsm/dukpt_cli/*.c $(LOCAL_PATH)/hsm/dukpt_common/*.c $(LOCAL_PATH)/hsm/pkcrypt_cli/*.c)
	LOCAL_SRC_FILES := $(ALL_FILES)
	LOCAL_LDLIBS +=  -llog -ldl
	include $(BUILD_SHARED_LIBRARY)
	
---

解析：    

LOCAL_PATH（当前目录）用于返回当前目录  my-dir通常为当前Android.mk所在目录 此变量不会被CLEAR_VARS清除，所以每个Android.mk文件只需要定义一次就可以了
	
CLEAR_VARS（变量清除）作用是清除模块变量（在include $(CLEAR_VARS)和include $(BUILD_XXX)之间的LOCAL_XXX模块变量），`当然LOCAL_PATH除外`

LOCAL_SRC_FILES （源码文件）不过也不必列出`头文件`和`包括文件`

BUILD_SHARED_LIBRARY（动态库编译）表示编译成动态库 还有BUILD_STATIC_LIBRARY，和BUILD_SHARED_LIBRARY类似，表示编译成静态库，静态库不会被拷贝到APK中。

LOCAL_LDLIBS 其他模块变量LOCAL_LDLIBS（链接库）用于额外链接选项，所有的库都有“-l”前缀。可同时列出多个库，用空格隔开，例如：` LOCAL_LDLIBS := -llog -ldl`

LOCAL_SHARED_LIBRARIES:  表示模块在运行时要依赖的共享库（动态库），在链接时就需要，以便在生成文件时嵌入其相应的信息。

LOCAL_STATIC_LIBRARIES: 表示该模块需要使用哪些静态库，以便在编译时进行链接。


### 2.Application.mk 一个例子

Application.mk文件是`可选`的.默认情况下，如果ndk-build命令找不到Application.mk文件的话，就会使用如下规则进行编译：

1）会编译全部在Android.mk文件中列出的模块；

2）对于所有模块，NDK编译系统会根据“armeabi” ABI来生成机器代码，即指令集是ARMv5TE。

`具体来说，Application.mk文件中，可以包含对下面几个变量的定义`

APP_PROJECT_PATH   这个变量会告诉编译系统，你应用程序项目的根目录的绝对路径。

PP_MODULES  这个变量是可选的。如果APP_MODULES变量没有被定义的话，NDK将编译在Android.mk文件中定义的所有模块，以及所有其包含的子MakeFile文件。
如果APP_MODULES变量被定义了的话，则NDK只会编译明确列出的这几个模块。注意，所有模块的名字要以空格分割，并且NDK会自动计算每个模块之间的依赖关系。

APP_OPTIM  这个变量是可选的，可以被定义成“release”或"debug"，用来告诉NDK将所有的模块编译成Release版本还是Debug版本。

APP_CFLAGS  如果想在编译所有模块的C或者C++源码时，都需要指定一些特殊的编译选项的话，可以通过定义这个变量实现。



### 3.makefile中"=" 及":="的区别

> 1、“=”

make会将整个makefile展开后，再决定变量的值。也就是说，变量的值将会是整个makefile中最后被指定的值。看例子：

        x = foo
        y = $(x) bar
        x = xyz

在上例中，y的值将会是 xyz bar ，而不是 foo bar 。

> 2、“:=”

“:=”表示变量的值决定于它在makefile中的位置，而不是整个makefile展开后的最终值。

        x := foo
        y := $(x) bar
        x := xyz

在上例中，y的值将会是 foo bar ，而不是 xyz bar 了。
      
### 附录参考

1.[Android NDK编译选项设置](http://blog.csdn.net/crash163/article/details/52228412)

2.[Android.mk详解](http://blog.csdn.net/ly131420/article/details/9619269)

3.[android.mk](http://android.mk/)

4.[ Android.mk的用法和基础 && m、mm、mmm编译命令 ](http://blog.csdn.net/zhandoushi1982/article/details/5316669)

5.[Application.mk语法解释 ](http://blog.csdn.net/roland_sun/article/details/46318893)

---

<p> **如果有疑问或者写的不对的地方,还请大家留言指出来..** </p>