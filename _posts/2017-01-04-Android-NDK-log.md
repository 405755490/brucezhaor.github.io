---
layout: post
title:      "[转]Android下LOG打印"
subtitle:   "各种LOG打印"
tags:       [LOG ,NDK]
mathjax:    false
date:       2017-01-04
author:     "Undefined"
description: ""
header-img: "/img/tags-bg.jpg"
categories:  [NDK]

---

## 目录
{: .no_toc}

* 目录
{:toc}


### `一:在源码开发模式下`

#### 1:包含头文件:

	#include <cutils/log.h>  

#### 2:定义宏LOG_TAG

    #define LOG_TAG "MY LOG TAG"  

#### 3:链接log对应的.so库

在Android.mk文件中加入如下语句:

    LOCAL_SHARED_LIBRARIES +=\  
            libcutils  
            
接下来就可以直接使用LOGD来打印log信息了.

### `二:在NDK开发模式下`

#### 1:包含头文件:

    #include <android/log.h>  

#### 2:定义宏LOG_TAG

    #define LOG_TAG "MY LOG TAG"   
    #define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, LOG_TAG, __VA_ARGS__)  
	#define LOGE(fmt,args...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG,fmt,##args)
	

	

#### 3:链接log对应的.so库

在Android.mk文件中加入如下语句:

    LOCAL_LDLIBS := -llog  

接下来就可以直接使用LOGD来打印log信息了.

### `三：在Java代码中`

#### 1:导入包

    import android.util.Log;  

#### 2：使用

    private static final String TAG = "your_tag";  
    Log.d(TAG,"show something");  

在程序运行过程中可以通过`adb shell`下的logcat指令看到相应的内容。或在`Eclipse下的ADT的LogCat窗口`中看到相应的内容了.


### `四：#、##、...和__VA_ARGS__ 用法`

> **`宏定义中的特殊参数(#、##、...和__VA_ARGS__) `**

	{% highlight c%}
	#define CHECK1(x, ...) if (!(x)) { printf(__VA_ARGS__); } 
	#define CHECK2(x, ...) if ((x)) { printf(__VA_ARGS__); }  
	#define MACRO(s, ...) printf(s, __VA_ARGS__)  
	 
	CHECK1(0, "here %s %s %s", "are", "some", "varargs1(1)\n"); //here are some varargs1(1)
	CHECK1(1, "here %s %s %s", "are", "some", "varargs1(2)\n");   // won't print
	CHECK2(0, "here %s %s %s", "are", "some", "varargs2(3)\n");   // won't print  
	CHECK2(1, "here %s %s %s", "are", "some", "varargs2(4)\n"); //here are some varargs2(4)
	MACRO("hello, world\n"); //hello, world
	{% endhighlight c%}
	
---
	
> **`## 替换前面的变量`**

#define LOGE(fmt,args...) __android_log_print(ANDROID_LOG_ERROR, LOG_TAG,fmt,##args)

##args替换参数的args. 如果省略##， 那么LOGE至少要`传递两个参数`，加上##，可以只有一个参数..

	{% highlight c%}
	#define LOG0(fmt,args...) __android_log_print(ANDROID_LOG_ERROR, "CPay",fmt,args)
	#define LOG1(fmt,args...) __android_log_print(ANDROID_LOG_ERROR, "CPay",fmt,##args)
	#define LOG2(fmt ,...) __android_log_print(ANDROID_LOG_ERROR, "CPay",fmt, __VA_ARGS__)
	#define LOG3(...) __android_log_print(ANDROID_LOG_ERROR, "CPay",__VA_ARGS__)

	//LOG0(" Welcome to C/C++");
	LOG0(" Welcome to C/C++[%d]" ,1);

	LOG1(" Welcome to C/C++");
	LOG1(" Welcome to C/C++[%d]" ,1);

	//LOG2(" Welcome to C/C++");
	LOG2(" Welcome to C/C++[%d]" ,2);

	LOG3(" Welcome to C/C++");
	LOG3(" Welcome to C/C++[%d]" ,3);
{% endhighlight c%}



---

### `五.LOGE中十六进制打印`

BCD码的打印方法:`可以在C/C++代码中转换成 字符串 来打印` BcdToAscii()

{% highlight c%}
void BcdToAscii (u8 *ascii_buf , u8 *bcd_buf ,int conv_len ,u8 type )
{

	int  cnt ;
	u8 *pBcdBuf;

	pBcdBuf = (u8 *)bcd_buf;

	if ( (conv_len & 0x01) && type ) /*判别是否为奇数以及往那边对齐*/
	{                           /*0左，1右*/
		cnt = 1 ;
		conv_len ++ ;
	}
	else
		cnt = 0 ;
	for ( ; cnt < conv_len ; cnt ++ , ascii_buf ++ ) 
	{
		*ascii_buf = ( ( cnt & 0x01 ) ? ( *pBcdBuf ++ & 0x0f ) : ( *pBcdBuf >> 4 ) ) ;
		*ascii_buf += ( ( *ascii_buf > 9 ) ? ( 'A' - 10 ) : '0' ) ;
	}
	*ascii_buf=0;
	return ;
}
{% endhighlight c%}







