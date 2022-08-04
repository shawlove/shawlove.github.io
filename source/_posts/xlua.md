---
title: xlua详解
date: 2022-07-27 09:41:07
tags:
coverImg:
categories:
---

# 为什么要用lua做热更新
lua是解释性语言（不事先编译，运行时通过lua解释器跨平台的编译代码）。  
每次代码更新时，将脚本文件作为资源下载下来，便可以在不重新打包的情况下更新业务逻辑。

# C#与lua为什么能交互
* lua提供了一系列C接口用于和C语言交互
* lua维护一个虚拟栈，C接口可以操作虚拟栈
* lua_CFunction机制可以让lua调用C方法
* 注册表机制
* light userdata和full userdata机制
* C#有与原生代码的互操作性（P/Invoke），可以与非托管代码互相操作，如C代码。  

lua-c交互：<https://shawlove.github.io/2022/08/02/lua-C%E4%BA%A4%E4%BA%92/>  
p/Invoke：<https://shawlove.github.io/2022/08/03/C-NativeInteroperability/>

# LuaCallCSharp
XLua用[LuaCallCSharp]标记lua端需要调用的C#代码，并为其生成代码。接下来进行原理分析。  
lua调用C#的情况：  
1. 调用类型的static字段、static属性、static方法、const字段（CS.CSClass.StaticOrConst）
2. 创建C#对象（CS.CSClass()）
3. C#端主动传值到lua

1，2两条XLua通过封装CS表使对象控制权在C#端，所以lua操作的所有C#对象都可以在C#端事先封装。  
## 封装CS对象
需要用到lua的三个特性：
* lua_CFunction  
C#对象的属性、字段、方法都封装成lua_CFunction,C#端的定义如下  
![](https://pic.imgdb.cn/item/62e9dbe516f2c2beb1c0cac1.png)  
[UnmanagedFunctionPointer]指定CallingConvention。这里设置为Cdecl是因为lua_CFunction默认是Cdecl。设置了CallingConvention后这里定义的lua_CSFunction在native code中就和lua_CFunction一致了。  
下面是生成代码的例子，我们来分析一下
![](https://pic.imgdb.cn/item/62e9de2216f2c2beb1c30c48.png)
[MonoPInvokeCallbackAttribute]用于标记作为非托管代码回调的方法。  
`ObjectTranslator`与一个lua_State对应。  
`FastGetCSObj`根据index获取cs对象，这里index为1的原因是第一个参数是self。1代表栈底、lua_CFunction调用期间使用的是单独的栈，每次调用前清空、参数按顺序压入栈。所以栈底就是第一个参数。  
接下来获取剩余参数调用方法，结果压入栈即可。
* userdata  
代表一个CS对象，为其设置metatable实现对CS对象的访问。当日metatable的内容就是我们封装的属性、字段、方法。userdata的地址作为key，缓存到C#字典中。
所以上图中的FastGetCSObj就是通过userdata的地址获取缓存CS对象。这样也可以保证CS对象不被GC
* lua注册表  
每种类型的CS对象都有对应的metatable，将metatable缓存到lua注册表中。key值用`luaL.Ref()`获取，并在C#缓存。
## 值类型GC问题
值类型如果走userdata地址作为key缓存对象这一套会有一个装箱过程。  
解决方法：基于基元类型或由基元类型组成的结构体，复制一份到lua的userdata中。获取时按照字段排布顺序复制到C#。（在Xlua中对应Pack，UnPack方法，会为加了GCOptimizeAttribute点结构体生成对应代码）。
# CSharpCallLua
C#调用lua的情况：
1. 主动获取lua的值
2. lua调用C#端回调，传入lua值

lua能获取到的C#端回调，都是在C#端封装成的lua_CFunction。所以本质还是C#主动获取lua的值。重点是function、table。  
## function
也就是把function映射到委托，前提是委托标记了[CSharpCallLua]并为其生成了代码（XXXBridge）。例子如下：
![](https://pic.imgdb.cn/item/62ea493416f2c2beb1415563.png)  
![](https://pic.imgdb.cn/item/62ea4a0216f2c2beb142527e.png)
![](https://pic.imgdb.cn/item/62ea497816f2c2beb1419f96.png)  
首先通过`luaL_Ref()`和function在注册表中获取一个key作为function的引用，也就是上图中的luaReference。function为key，引用为value和引用为key，function为value都写入lua注册表中。然后在C#创建XXBridge，并用刚刚得到的引用作为key存到字典中。这样获取一个function时可以直接找到对应的Bridge，然后通过上图的GetDelegeteByType获取委托。  
这个委托的内容就是上图的`_Gen_Delegete_Imp0()`。  
现在来分析一下这个方法。  
`pcall_prepare()`的作用是将对应function放到栈顶(-1)，错误处理方法放到-2。压入参数后，调用function，最后把两个方法弹出栈。如果有返回值，在执行后获取即可。  
将function放到注册表可以避免被GC。  
当function不被C#引用时清空注册表中对应function相关信息。清空的时机：  
上面说的XXBridge的字典，value是WeakRefrence。当C#未引用function时，Bridge会被gc，在gc前执行清空注册表信息即可。
## table
基元类型、枚举、数组和字典不讨论比较简单，结构体在上面已经讨论过了。
### 映射到接口 
生成Bridge类，继承实现对应接口。  
获取时，new一个Bridge，获取引用，写入注册表。
使用时，见下方例子。  
![](https://pic.imgdb.cn/item/62eb2c7f16f2c2beb111ac90.png)  
通过引用把table压入栈顶，然后通过方法名、属性名取值。如果是function，压入参数，调用function，获取返回值。如果是其他值，获取即可。
### 映射到类
类不确定能否继承，所以只能按字段从table中获取然后赋值。所以映射到类后，table与其并无联系。而映射到接口是有引用链接的。