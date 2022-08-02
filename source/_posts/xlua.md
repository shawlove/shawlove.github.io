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
* C#有与原生代码的互操作性（P/Invoke），可以操作非托管代码，如C代码。  

lua-c交互：  

# lua调用C#
## 调用C#对象的属性、字段、方法
在C#端把C#对象的属性、字段、方法都封装成lua_CFunction。传入lua端时设置userdata的metatable，使lua调用userdata时会调用C#端的方法。
## 创建C#对象
在C#创建CS表，封装构造方法  
# C#传值到lua
1. 通过传入顺序作为key储存到字典中（防止C#端无引用造成gc，也用于通过userdata地址查询C#对象）
2. 根据类型获取metatable（没有则创建，metatable存在lua注册表中）
3. 在lua端创建userdata，大小为sizeof(int)，地址与key保持相同
4. 设置metatable
## 创建metatable
把C#对象的属性、字段、方法都封装成lua_CFunction，通过` Marshal.GetFunctionPointerForDelegate(function)`获取委托指针，设置到metatable中。  
通过`luaL_ref()`获取注册表中未使用的key，保存到lua注册表中。这里的key也就是xlua中的type_id。  
如何封装属性、字段、方法呢？答案是生成代码。所以用[LuaCallCSharp]标记需要生成代码的类。
## 创建CS表
通过命名空间创建树状的表（CS是张表，CS.XX也是张表），存到lua注册表中。  
类型的静态变量，常量也会写入到对于的表中。
## 值类型GC问题
xlua提供了[GCOptimize]处理值类型GC。  
如果不加这个特性，会默认按照object的方式传值，会有一次装箱操作。  
原理： 基于基元类型或由基元类型组成的结构体，复制一份到lua的userdata中。获取时按照字段排布顺序复制到C#。（在Xlua中对应Pack，UnPack方法，会为加了GCOptimizeAttribute点结构体生成对应代码）。

# C#获取Lua的值
也就是官方文档说的映射
## 接口