---
title: GLFW API 汇总
tags: [OpenGL,C++,GLFW]
categories: API
date: 2022-01-10 11:01:47
---
学完之后不常用，老是会忘，此文档用于帮助恢复记忆。且会不断维护。    
官方文档:<https://www.glfw.org/docs/latest/>
# glfwInit & glfwTerminate
初始化GLFW，多次调用不会重复初始化（如果已经初始化会直接返回GLFW_TRUE）。  
初始化成功后，退出程序时必须调用glfwTerminate
```c++
if(!glfwInit())
{
    //Handle initialization failure
}

...

glfwTerminate()
```

# glfwWindowHint
为下次调用glfwCreateWindow设置hint  
Window creation hints有很多，官方文档：<https://www.glfw.org/docs/latest/window_guide.html#window_hints>  
常用的：
* GLFW_CONTEXT_VERSION_MAJOR  
* GLFW_CONTEXT_VERSION_MINOR  

指定OpenGL的版本 如4.3  
```c++
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
```

# glfwCreateWindow & glfwDestroyWindow
创建一个glfw窗口,并关联了OpenGL的上下文
![](https://pic.imgdb.cn/item/61dbaa0a2ab3f51d9171cd47.png)
share可以让两个窗口共享数据（textures,vertex and element buffers,etc.）

程序退出时调用glfwDestroyWindow(window)

# glfwMakeContextCurrent
将指定window的OpenGL上下文作为当前线程的上下文  
一个线程只能有一个上下文  
更多细节：<https://www.glfw.org/docs/3.3/group__context.html#ga1c04dc242268f827290fe40aa1c91157>

# glfwSwapBuffers
交换指定window的前后缓冲区  
GLFW窗口默认是双缓冲，每次渲染指令完成后，调用此方法交换缓冲区指针。显示已经渲染好的画面。 如果是单缓冲，会出现闪屏的现象。因为屏幕上的画面不是已经绘制完成的，而是从左到右、从上到下逐像素绘制的

# glfwSwapInterval
![](https://pic.imgdb.cn/item/61dbcb132ab3f51d91897c8b.png)
设置OpenGL交换缓冲区时间间隔：调用glfwSwapBuffers会等待屏幕刷新interval次后返回  
上述行为也叫做垂直同步（vertical synchronization,vertical retrace synchronization,vsync）  

# glfwWindowShouldClose
返回指定窗口的close flag

# glfwPollEvents
处理事件，渲染循环中调用即可

# glfwGetTime
除非使用glfwSetTime来设置时间，否则它会返回自GLFW初始化以来所经过的时间。