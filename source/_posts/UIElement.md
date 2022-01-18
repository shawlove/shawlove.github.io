---
title: UIElement / UI Toolkit
date: 2022-01-17 10:11:36
tags: [Unity,UIElement,UI Toolkit]
categories: UIElement,UI Toolkit
---

# 前言
UI Toolkit以前叫UIElement，是一个东西。  
Unity有三个UI系统
* UI Toolkit （runtime和editor下可使用，不过runtime在preview阶段不推荐实际使用）
* Unity UI(UGUI) （runtime下使用）
* Immediate Mode GUI(IMGUI)  （editor下使用）

三个系统具体比较：<https://docs.unity3d.com/Manual/UI-system-compare.html>  
原文：<https://docs.unity3d.com/Manual/UIElements.html>
# UI system
由以下几个特性组成
* Visual tree： 由多个轻量级节点组成的图，代表一个UI
* Controls： 一个UI控制组件库（按钮，弹窗等等），可以自定义
* Data bingding system： 链接属性和控件
* Layout Engine： 基于css，定义ui元素的布局，样式
* Event System： 处理用户交互（触碰、点击等等） The system includes a dispatcher, a handler, a synthesizer, and a library of event types.
* UI Renderer： 直接构建在gpu上的渲染系统
* UI Toolkit Runtime Support (via the UI Toolkit package): runtime ui，现在在preview阶段

## The Visual Tree
轻量级节点指VisualElement。  
一个例子:  
![](https://pic.imgdb.cn/item/61e679062ab3f51d91055a11.png)  
### Visual elements
VisualElement 是所有节点的基类，包含一些通用属性（styles，layout data，event handlers）。这些属性提供给controls使用。  
VisualElement可以有子节点父节点  
### Panels
panel 是 visual tree的父物体。visual tree只有连接到panel上才能渲染内部的visual element。  
panel属于window，比如Editor Window。  
panel处理foucus control（窗口聚焦）和visual tree的事件发送  
每个element都有对panel的直接引用，没有连接返回null
### Draw order
渲染顺序依据DFS（深度优先搜索）  
子element渲染在父elments上面  
![](https://pic.imgdb.cn/item/61e67b2c2ab3f51d91079cf3.png)  
用以下方法可以改变渲染顺序  
* BringToFront()
* SendToBack()
* PlaceBehind()
* PlaceInFront()
### Coordinate and position systems
UI Toolkit自动计算element的位置和尺寸（通过element.style.layout,类型为Rect）  
UI Toolkit有两种坐标：
* Relative：相对于已经计算的element（有相同父节点的element）位置
* Absolute：相对父节点的位置。不会影响relative节点都计算  

使用方法：
```c#
var newElement = new VisualElement();
newElement.style.position = Position.Relative;
newElement.style.left = 15;
newElement.style.top = 35;
```

## The Layout Engine
layout engine通过element.style布置节点。layout engine使用Yoga的布局原则，Yoga实现了Flexbox的一部分功能。Flexbox是一个HTML/CSS的布局系统  
Yoga文档：<https://yogalayout.com/docs>  
Flexbox文档：<https://css-tricks.com/snippets/css/a-guide-to-flexbox/>  

### Behavior
默认情况，所有element都是layout对一部分。layout有以下几个默认行为：
* 一个容器纵向分配子节点
* 容器的rect包含所有子节点的rect，可以被其他布局属性限制
* 文本节点会用显示文本的尺寸计算节点的尺寸，可以被其他布局属性限制
