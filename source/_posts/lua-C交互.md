---
title: lua-C交互
date: 2022-08-02 14:57:28
tags:
coverImg:
categories:
---
lua本身由C编写，同时也提供了与C交互的接口。  
# lua_State
代表一个lua的一个线程，lua的所有C接口的第一个参数是lua_State（除了lua_newstate,他是创建一个lua_State）  
每个lua线程都维护一个虚拟栈和一个lua_CFunction栈（在调用lua_CFunction时，C接口操作的是lua_CFunction栈，其他时候操作的虚拟栈）
# 栈索引
-1代表栈顶，1代表栈底
![](https://pic.imgdb.cn/item/62e792fe8c61dc3b8ee77b34.png)  

# lua_CFunction
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

# 注册表
lua事先创建了一个张表，提供给C代码使用。这张表并不在虚拟栈中，但是可以由一个假索引（pseudo-index）LUA_REGISTRYINDEX获取。  
常见用途是储存userdata的metatable  
![](https://pic.imgdb.cn/item/62e8cf8716f2c2beb1d15283.png)  
luaL_系列的接口多是与注册表相关的API

# userdata
light userdata代表指针，lua不关心其生命周期  
full userdata代表一快由C控制的内存，可以通过lua_newuserdata创建指定字节大小的userdata  
为userdata设置metatable后，lua调用userdata时会跳转到C代码。 
   
`以上，实现了lua-C的交互`