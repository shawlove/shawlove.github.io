---
title: OpenGL API 汇总
tags: OpenGL
categories: API
date: 2022-01-10 10:58:42
---
学完之后不常用，老是会忘，此文档用于帮助恢复记忆。且会不断维护。  
官网文档:<https://www.khronos.org/registry/OpenGL-Refpages/gl4/>

# glClearColor 
![](https://pic.imgdb.cn/item/61dbcf872ab3f51d918cf948.png)
指定清理缓冲区时的颜色

# glClear
![](https://pic.imgdb.cn/item/61dbd02b2ab3f51d918d7f4d.png)
用预设的值（颜色）清理缓冲区  
mask：
* GL_COLOR_BUFFER_BIT 颜色缓冲区
* GL_DEPTH_BUFFER_BIT 深度缓冲区
* GL_STENCIL_BUFFER_BIT 模板缓冲区
masks的按位或运算指定清理那些缓冲区（GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT|GL_STENCIL_BUFFER_BIT）

# glCreateShader
![](https://pic.imgdb.cn/item/61dbd2c22ab3f51d918fba41.png)
创建指定类型的空shader（没有内容），返回指针  
shader需要指定源代码（用glShaderSource）  

shaderType：
* GL_COMPUTE_SHADER 计算着色器
* GL_VERTEX_SHADER 顶点着色器
* GL_TESS_CONTROL_SHADER 曲面细分着色器 control阶段
* GL_TESS_EVALUATION_SHADER 曲面细分着色器 evaluation阶段
* GL_GEOMETRY_SHADER 几何着色器
* GL_FRAGMENT_SHADER 片段着色器

# glShaderSource
![](https://pic.imgdb.cn/item/61dbd5772ab3f51d9191c76a.png)
指定shader代码  
count指定string的长度  
string是个字符串数组  
length指定对应索引的字符串长度，为null代表字符串都以null结尾

# glCompileShader
![](https://pic.imgdb.cn/item/61dbdd1a2ab3f51d9197d5ce.png)
编译指定shader  
编译状态会存在shader中（GL_TRUE,GL_FALSE），可通过glGetShader获取  
编译log（包括错误日志）也会存在shader中，通过glGetSahderInfoLog获取  

# glCreateProgram
![](https://pic.imgdb.cn/item/61dbdd412ab3f51d9197faa1.png)
创建一个空program，返回指针  
program可以用glAttackShader，将shader附加到program上  
成功attach shader，成功编译shader，成功链接program（glLinkProgram）后，program中会创建一个或多个可执行程序  
这个程序不会立即执行，用glUseProgram后执行  
glDeleteProgram可以删除这个program  

# glAttachShader
![](https://pic.imgdb.cn/item/61dbdd5d2ab3f51d91980ffc.png)
附加到program上

# glLinkProgram
![](https://pic.imgdb.cn/item/61dbe11f2ab3f51d919bb61f.png)
链接program  
链接成功失败（GL_TRUE,GL_FALSE）可通过glGetProgram获取  
链接成功后，所有user-defined的uniform变量初始化为0，这个程序的所有uniform变量分配一个location，可以通过glGetUniformLocation获取  

# glUseProgram
![](https://pic.imgdb.cn/item/61dbe1562ab3f51d919be72e.png)
运行program

# glGenVertexArrays
![](https://pic.imgdb.cn/item/61dbe1c32ab3f51d919c4df4.png)
创建一个VAO数组  
VAO是组织缓冲区（VBO）的方法

# glBindVertexArray
![](https://pic.imgdb.cn/item/61dbf5db2ab3f51d91afafcb.png)
绑定VAO  
绑定后缓冲区会与和这个VAO关联  
array==0：取消当前绑定的VAO

# glDrawArrays
![](https://pic.imgdb.cn/item/61dbf2672ab3f51d91abcdce.png)
开始渲染（enabled arrays：VAO）  

# glGetShaderiv
![](https://pic.imgdb.cn/item/61e0efaa2ab3f51d91097c4c.png)
获取shader的参数  
pname:
* GL_SHADER_TYPE 返回shader类型（GL_VERTEX_SHADER | GL_GEOMETRY_SHADER | GL_FRAGMENT_SHADER ）
* GL_DELETE_STATUS 返回shader是否被delete
* GL_COMPILE_STATUS 返回是否编译成功
* GL_INFO_LOG_LENGTH 返回log字符串长度
* GL_SHADER_SOURCE_LENGTH 返回source的字符串长度，没有返回0

# glGetShaderInfoLog
![](https://pic.imgdb.cn/item/61e0f0f22ab3f51d910a7b3b.png)
返回shader的log

# glGetError
![](https://pic.imgdb.cn/item/61e0f2a32ab3f51d910bc09e.png)
获取错误flag  
每次调用glGetError会返回一个随机error flag，然后清理这个flag，所以如果有多个error flag需要在循环中调用，直到glGetError返回GL_NO_ERROR  
errors：
* GL_NO_ERROR 没有错误
* GL_INVALID_ENUM 向枚举参数传了一个未知枚举
* GL_INVALID_VALUE 数值超出范围
* GL_INVALID_OPERATION 当前不允许的操作
* GL_INVALID_FRAMEBUFFER_OPERATION framebuffer未完成
* GL_OUT_OF_MEMORY 内存爆了
* GL_STACK_UNDERFLOW 栈下溢
* GL_STACK_OVERFLOW 栈溢出

# glGetUniformLocation 
![](https://pic.imgdb.cn/item/61e1263a2ab3f51d91367378.png)
传入uniform变量名，返回其location

# 