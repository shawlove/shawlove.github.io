---
title: 计算机图形学编程（使用OpenGL和C++）第二章
tags: [OpenGL,C++]
cover: 'https://pic.imgdb.cn/item/61d7e8c72ab3f51d91f4be22.jpg'
top_img: 'https://pic.imgdb.cn/item/61d7e8c72ab3f51d91f4be22.jpg'
categories: 计算机图形学编程（使用OpenGL和C++）
date: 2022-01-07 17:02:07
---
# OpenGL图像管线
OpenGL是整合软硬件的多平台2D和3D图像API  
硬件：提供多级图形管线，用GLSL进行编程  
软件：API由C编写，所以直接兼容C和C++。对于其他语言，都有对应的封装库，性能几乎相同  
# OpenGL管线
管线流程：
顶点着色器->曲面细分着色器->几何着色器->光栅化->片段着色器->像素操作  
顶点处理 -> 图元处理 -> 片段处理 -> 隐藏面消除等  
![](https://pic.imgdb.cn/item/61d802a62ab3f51d910d0dcb.png)  
可用GLSL编程的阶段：顶点着色器、曲面细分着色器、几何着色器、片段着色器  
C++程序会将GLSL代码加载到对应阶段
## C++/OpenGL应用程序
创建一个GLFWwidnow，并设置背景色

![](https://pic.imgdb.cn/item/61d809062ab3f51d91159c62.png)

完整代码
```c++
#include <GL\glew.h>
#include <GLFW\glfw3.h>
#include <iostream>

using namespace std;

init(GLFWwindow* window);

void init(GLFWwindow* window){} //暂时还没有内容

void display(GLFWwindow* window,double currentTime){
    //设置清除缓冲区的颜色
    glClearColor(1.0,0.0,0.0,1.0);
    //清除后缓冲区
    glClear(GL_COLOR_BUFFER_BIT);
}

int main(void){
    //初始化GLFW库
    if (!glfwInit()) { exit(EXIT_FAILURE); }
    //设置OpenGL的版本 4.3
    //glfwWindowHint用于设置OpenGL
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    GLFWwindow* window = glfwCreateWindow(600, 600, "Chapter2 - programl", NULL, NULL);
    //为创建的窗口设置上下文 每个线程只有一个上下文
    glfwMakeContextCurrent(window);
    //初始化GLEW库
    if (glewInit() != GLEW_OK) { exit(EXIT_FAILURE); }
    //开启垂直同步 关闭0
    glfwSwapInterval(1);
    
    init(window);

    //关闭窗口后退出循环
    while (!glfwWindowShouldClose(window)) {
        //自定义函数，渲染指令 
	    display(window, glfwGetTime()); 
        //GLFW窗口默认是双缓冲，每次渲染指定完成后，交换缓冲区指针。显示已经渲染好的画面。 如果是单缓冲，会出现闪屏的现象。因为屏幕上的画面不是已经绘制完成的，而是从左到右、从上到下逐像素绘制的
		glfwSwapBuffers(window); 
        //处理事件，如点击
		glfwPollEvents();
	}

    //销毁窗口
    glfwDestroyWindow(window);
    //终止glfw
    glfwTerminate();
    exit(EXIT_SUCCESS);
}
```

## 顶点着色器和顶点着色器
构建三角形：
```c++
//mode:图元的类型 三角形、点
//first:那个顶点开始
//count：数量
//调用此方法后 会开始运行GLSL代码
glDrawArrays(GLenum mode, Glint first,GLsizei count)
```

硬编码显示一个点  
![](https://pic.imgdb.cn/item/61d80b742ab3f51d9117a928.png)
```c++
(#include与之前相同)
#define numVAOs 1

GLuint renderingProgram;
Gluint vao[numVAOs];

GLuint createShaderProgram(){
    const char *vshaderSource=
    "#version 430 \n"
    "void main(void) \n)"
    //gl_Position是预定义的输出变量
    "{ gl_Position = vec4(0.0,0.0,0.0,1.0); }";
    
    const char *fshaderSource=
    "#version 430 \n"
    "out vec4 color; \n"
    "void main(void) \n"
    "{ color = vec4(0.0,0.0,1.0,1.0) }";
    //创建顶点和片段着色器，返回地址
    GLuint vShader = glCreateShader(GL_VERTEX_SHADER);
    GLuint fShader = glCreateShader(GL_FRAGMENT_SHADER);
    //指定脚本
    glShaderSource(vShader,1,&vshaderSource,NULL);
    glShaderSource(fShader,1,&fshaderSource,NULL);
    //编译
    glCompileShader(vShader);
    glCompileSHader(fShader);
    
    //创建程序
    GLuint vfProgram = glCreateProgram();
    //将着色器加入程序
    glAttackShader(vfProgram,vShader);
    glAttackShader(vfProgram,fShader);
    //与OpenGL链接
    glLinkProgram(vfProgram);
    return vfProgram
}

void init(GLFWwindow* window)
{
    renderingProgram = createShaderProgram();
    //vao ：Vertex Array Object 顶点数组对象，第四章会讲
    glGenVertexArrays(numVAOs,vao);
    glBindVertexArray(vao[0])
}

void display(GLFWwindow* window, double currentTime)
{
    //将着色器加载进硬件 而不是直接运行
    glUseProgram(renderingProgram);
    glDrawArrays(GL_POINTS,0,1);    
}

...main()函数与之前相同
```

## 曲面细分着色器
12章具体介绍，4.0版本后加入的功能。用于生成大量三角形，通常是网格形式。同时也可以操作这些三角形

## 几何着色器
一次操作一个图元（最通用的图元是三角形） 可以让图元变形，比如拉伸、缩小、删除、增加。是一种将简单模型转化为复杂模型的方法

## 光栅化
3D世界中的点、三角形、颜色等全部需要展现在一个2D显示器上。这个2D屏幕由光栅——矩形像素阵列组成  
3D物体光栅化后，OpenGL将物体中的图元转换成片段。光栅化过程确定了用于显示图元的所有像素绘制的位置。  
开始时先对三角形（图元）的每对定点进行插值，这个过程可以通过选项调节。一般是线性插值

## 片段着色器
片段着色器用于为光栅化的像素指定颜色  
gl_FragCoord 访问输入片段的坐标  
例子
```c++
#version 430
out vec4 color
void main(void)
{
    if(gl_FragCoord.x<200)
        color=vec4(1.0, 0.0, 0.0, 1.0);
    else
        color=vec4(1.0, 0.0, 1.0, 1.0);
}
```
![](https://pic.imgdb.cn/item/61d80bb22ab3f51d9117e367.png)
## 像素操作
隐藏面消除（Hidden Surface Removal，HSR)  
此阶段不可编程，可配置  
工作原理：  
通过两个缓冲区完成：颜色缓冲区、深度缓冲区（Z缓冲、Z-buffer）。这两个缓冲区大小和光栅相同（屏幕的每个像素，在缓冲区都有对应的）  
1. 在每个场景渲染前，深度缓冲区的值都为最大深度值  
2. 片段着色器输出像素时，计算与观察者的距离  
3. 留下距离近的像素  
# 检测OpenGL和GLSL错误
* glGetError获取当前OpenGL错误
* glGetShaderiv、glGetShaderInfoLog获取shader错误log
* glGetProgramiv、glGetProgramInfoLog获取program错误log  
可封装出三个通用方法
```c++
void printShaderLog(GLuint shader) {
    int len = 0;
    int chWrittn = 0;
    char *log;
    glGetShaderiv(shader, GL_INFO_LOG_LENGTH, &len);
    if (len > 0) {
        log = (char *)malloc(len);
        glGetShaderInfoLog(shader, len, &chWrittn, log);
        cout << "Shader Info Log: " << log << endl;
        free(log);
    }
}

void printProgramLog(int prog) {
    int len = 0;
    int chWrittn = 0;
    char *log;
    glGetProgramiv(prog, GL_INFO_LOG_LENGTH, &len);
    if (len > 0) {
        log = (char *)malloc(len);
        glGetProgramInfoLog(prog, len, &chWrittn, log);
        cout << "Program Info Log: " << log << endl;
        free(log);
    }
}

bool checkOpenGLError() {
    bool foundError = false;
    int glErr = glGetError();
    while (glErr != GL_NO_ERROR) {
        cout << "glError: " << glErr << endl;
        foundError = true;
        glErr = glGetError();
    }
    return foundError;
}
```

使用方法  
```c++
  GLint vertCompiled;
    GLint fragCompiled;
    GLint linked;
    
    ...
    //检查编译错误
    glCompileShader(vShader);
    checkOpenGLError();
    //获取编译是否成功
    glGetShaderiv(vShader, GL_COMPILE_STATUS, &vertCompiled);
    if (vertCompiled == 1) {
         cout << "vertex compilation success" << endl;
    }
    else {
        cout << "vertex compilation failed" << endl;
        printShaderLog(vShader);
    }
    
    glCompileShader(fShader);
    checkOpenGLError();
    glGetShaderiv(fShader, GL_COMPILE_STATUS, &fragCompiled);
    if (fragCompiled == 1) {
        cout << "fragment compilation success" << endl;
    }
    else {
        cout << "fragment compilation failed" << endl;
        printShaderLog(fShader);
    }
    
    ...
    //检查链接错误
    glLinkProgram(vfprogram);
    checkOpenGLError();
    //获取链接是否成功
    glGetProgramiv(vfprogram, GL_LINK_STATUS, &linked);
    if (linked == 1) {
        cout << "linking succeeded" << endl;
    }
    else {
        cout << "linking failed" << endl;
        printProgramLog(vfprogram);
    }
```

图形调试器  
Nvidia显卡——Nsight  
Amd显卡——CodeXL  

# 从文件读取GLSL源代码
把glsl代码写到.glsl文件中，c++读取  

包含头文件:
```c++
#include <string>
#include <iostream>
#include <fstream>
```

读取：
```c++
string readFile(const char *filePath) {
    string content;
    ifstream fileStream(filePath, ios::in);
    string line = "";
    while (!fileStream.eof()) {
        getline(fileStream, line);
        content.append(line + "\n");
    }
    fileStream.close();
    return content;
}
```

使用：
```c++
GLuint createShaderProgram(){
    ...
    string vertShaderStr = readFile("vertShader.glsl");
    string fragShaderStr = readFile("fragShader.glsl");
    
    const char *vertShaderSrc = vertShaderStr.c_str();
    const char *fragShaderSrc = fragShaderStr.c_str();
    
    ...
}
```

# 从顶点构建对象
渲染一个三角形  
```c++
glDrawArrays(GL_TRIANGLES, 0, 3);
```

gl_VertexID代表顶点索引
```glsl
#version 430
void main(void)
{
	if (gl_VertexID == 0) gl_Position = vec4( 0.25,-0.25, 0.0, 1.0);
  else if (gl_VertexID == 1) gl_Position = vec4(-0.25,-0.25, 0.0, 1.0);
  else gl_Position = vec4( 0.25, 0.25, 0.0, 1.0);
}
```
![](https://pic.imgdb.cn/item/61e1108b2ab3f51d91227a71.png)
# 场景动画
通过给Shader赋值实现
给Shader变量赋值的机制叫做Uniform变量（统一变量）  
```c++
//获取Uniform变量  
GLuint offsetLoc = glGetUniformLocation(renderingProgram, "offset");
//赋值
glProgramUniform1f(renderingProgram, offsetLoc, x);
```  