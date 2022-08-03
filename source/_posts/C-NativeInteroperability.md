---
title: C# P/Invoke
date: 2022-08-03 11:31:33
tags:
coverImg:
categories:
---
CLR提供了两种机制和非托管代码互操作：
* Platform invoke，托管代码调用非托管代码函数
* COM interop，托管代码通过接口和COM交互
这篇文章只涉及Platform invoke（P/Invoke）
# 使用非托管DLL函数
![](https://pic.imgdb.cn/item/62ea198a16f2c2beb105b268.png)
如上，声明static extern方法，并用[DllImport]指定方法所在地址
## CallingConvention
指定非托管代码函数的调用约定。这里只讨论Cdecl和StdCall  
Cdecl:参数从右往左压入栈，调用者清理堆栈  
StdCall:参数从右往左压入栈，函数本身清理堆栈  
C++/C默认使用Cdecl，因为需要支持可变参数。如果是StdCall，函数本身不知道有几个参数入栈，所以无法清理堆栈。而Cdecl由调用者清理堆栈，调用者知道自己压入几个参数，所以函数结束能正确的弹出参数，清理堆栈。  
上图的情景是调用C函数，所以需要指定Cdecl。
# Interop Marshaling
Interop Marshaling处理函数参数与返回值在托管和非托管内存中的传递。Interop Marshaling是CLR（common language runtime）的marshalling service的运行时功能。  
大部分类型在托管和非托管内存中都有对应，见下图
![](https://pic.imgdb.cn/item/62ea3ceb16f2c2beb1324510.png)  
其中char和string的Marshal方式可以由[DllImport]的CharSet属性指定（ANSI还是Unicode，默认是ANSI）。  
委托默认被marshal成未托管函数指针。