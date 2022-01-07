---
title: 计算机图形学编程（使用OpenGL和C++）第一章
date: 2022-01-07 15:14:44
tags: OpenGL C++
cover: https://pic.imgdb.cn/item/61d7e8c72ab3f51d91f4be22.jpg
top_img: https://pic.imgdb.cn/item/61d7e8c72ab3f51d91f4be22.jpg
categories: 计算机图形学编程（使用OpenGL和C++）
---

# 1.1 语言和库
现代图像编程使用`图形库`完成  
有很多图形库，常见的平台无关的图形编程库叫做OpenGL（Open Graphics Library,开放图形库）  
在c++中使用OpenGL还需要其他库：  
* 窗口管理
* 扩展库
* 数学库
* 纹理管理
## 1.1.1 C++
介绍C++，跳过
## 1.1.2 OpenGL/GLSL
OpenGL 2.0版本引入OpenGL着色器语言（GLSL），3.1版本移除大量废弃功能（立即模式），强制使用着色器编程，4.0版本在可编程管线中加入细分阶段  
本书默认用户的显卡支持4.3版本的OpenGL
## 1.1.3 窗口管理
OpenGL不是把图像直接绘制到屏幕上，而是渲染到一个帧缓冲区，需要其他库来把缓冲区的内容绘制到屏幕的一个窗口中  
可选择库：CPW、GLOW、GLUI、GLFW  
本书使用GLFW
## 1.1.4 扩展库
Glee、GLLoader、GLEW、GL3W、GLAD
本书使用GLEW（OpenGL Extension Wrangler）
## 1.1.5 数学库
需要使用大量的向量和矩阵代数，所以需要一个数学库。常见库：Eigen、vmath。本书使用GLM（OpenGL Mathematics）
## 1.1.6 纹理管理
用于纹理加载，如FreeImage、DevIL、OpenGL Image（GLI）、GLraw
本书使用SOIL2
## 1.1.7 可选库
如模型加载库，本书实现一个简易模型加载器
# 1.2安装和配置
见书附录  
todo 记录安装过程