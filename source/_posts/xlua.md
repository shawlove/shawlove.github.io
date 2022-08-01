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

## lua栈索引
-1代表栈顶，1代表栈底
![](https://pic.imgdb.cn/item/62e792fe8c61dc3b8ee77b34.png)

## lua_CFunction
以下C方法为官方案例
```c
static int foo(lua_State *L){
    int n = lua_gettop(L); //参数个数
    lua_Number sum = 0.0;
    int i;
    for(i=1; i<=n ;i++>){
        if(!lua_isnumber(L,i)){
            lua_pushliteral(L, "参数错误，只能为number");
            lua_error(L);
        }
        sum += lua_tonumber(L,i);
    }
    lua_pushnumber(L, sum/n); //第一个返回值
    lua_pushnumber(L, sum); //第二个返回值
    return 2;
}
```

`static int MethodName(lua_State *L){}`是lua_CFunction的固定格式，只有这种格式的C方法才能被lua调用  
`lua_State`代表一个lua的一个线程，lua的所有方法的第一个参数是lua_State（除了lua_newstate,他是创建一个lua_State）  
`lua_gettop()`获取栈的高度。在调用lua_CFunction时，所有对虚拟栈的操作都会指向一个新的栈，姑且称为方法栈（方法栈在非lua_CFunction调用期间也保持激活）。方法栈在每次调用lua_CFunction会清空，所以这里栈的高度就是参数个数。  
`lua_isnumber(L,i)`指定索引处值是否number  
`lua_pushliteral(L,"")`字面量string入栈  
`lua_error()`报错，栈顶string作为错误信息  
`lua_tonumber(L,i)`指定索引处值转化成number  
`lua_pushnumber()`number入栈  

最后返回2代表有两个返回值，就是我们按顺序压入栈道两个number。方法的参数也是第一个参数先入栈，所以index=1处的值就是第一个参数。  
如上分析，这个函数就是计算所有参数的平均数和总和。

## full userdata

## lua注册表

## P/Invoke

如上，可以通过用C代码封装一部分lua的C接口提供给C#使用，C#就能操作lua了。  
而lua怎样操作C#呢？答案是C#对象作为userdata，并把所有属性、字段、方法都封装成lua_CFunction，放入userdata的metatable中。这样lua调用C#对象时都会回到C#端处理，这也是Xlua源码为什么都写在C#端的原因。  

# 如何组织C#和lua的交互以及注意的点
## c#对象传入lua
传入的情景
* 创建实例
* 获取静态变量
* c#调用lua方法，作为参数传入lua


`lua_newuserdata()`接口创建指定字节大小的full userdata，并返回地址。