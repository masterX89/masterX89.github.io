---
title: Android Studio常用快捷键
date: 2016-05-25 09:43:05
categories:
	- Android Studio
	- Android
tags:
	- Android Studio
	- Android
---

最近开始尝试转向Android Studio开发了，于是总结一下常用的快捷键。顺带着粘一段wikipedia上as的简介凑字数。

> Android Studio是一个为Android平台开发程序的集成开发环境。2013年5月16日在Google I/O上发布，可供开发者免费使用。
2013年5月发布早期预览版本，版本号为0.1。2014年6月发布0.8版本，至此进入beta阶段。第一个稳定版本1.0于2014年12月8日发布。
Android Studio基于JetBrains IntelliJ IDEA，为Android开发特殊定制，并在Windows、OS X和Linux平台上均可运行。


本篇适用于日常使用，详细快捷键请参考此篇[AndroidStudio快捷键汇总](http://seniorzhai.github.io/2015/02/05/AndroidStudio%E5%BF%AB%E6%8D%B7%E9%94%AE%E6%B1%87%E6%80%BB/)

### 重构-重命名 ###
- `Shift` + `F6`
这个很重要


### 提取局部变量 ###
- `Ctrl`(`Command`) + `Alt`(`Option`) + `V`</br>
![](http://i.imgur.com/XReDHEq.gif)

### 提取全局变量 ###
- `Ctrl`(`Command`) + `Alt`(`Option`) + `F`</br>
![](http://i.imgur.com/ztIJz8S.gif)

<!-- more -->

### 快速修复 ###
- `Alt`(`Option`) + `Enter`(用于类型强转等快速修复，不可以用于提取全局变量，和eclipse不一样)</br>
![](http://i.imgur.com/hanZPZS.gif)

### 全屏当前标签页 ###
- `Ctrl`(`Command`) + `Shift` + `F12`

### 撤销(undo) ###
- `Ctrl`(`Command`) + `Z`

### 重做(redo) ###
- `Ctrl`(`Command`) + `Shift` + `Z`

### 格式化代码 ###
- `Ctrl`(`Command`) + `Alt`(`Option`) + `L`

### 折叠代码 ###
- `Ctrl`(`Command`) + `.`</br>
![](http://i.imgur.com/Rvp4uTU.gif)

### 方法参数提示 ###
- `Ctrl`(`Command`) + `P`</br>
![](http://i.imgur.com/ZCZXeWc.gif)

### 显示注释文档 ###
- `Ctrl`(`Command`) + `Q`

### 生成构造器、Getter、Setter ###
- `Alt`(`Option`) + `Insert`

### 块选中 ###
- `Alt`(`Option`) + `J`（解放`Ctrl`(`Command`) + `LEFT`/`RIGHT`的福音）</br>
![](http://i.imgur.com/oSxvPVO.gif)

### 自动代码 ###
- `Ctrl`(`Command`) + `J`</br>
![](http://i.imgur.com/Mlqi2Z2.gif)

### 查询当前元素在工程中的引用 ###
- `Alt`(`Option`) + `F7`</br>
![](http://i.imgur.com/jRAcBAQ.gif)

### 方法间快速跳转 ###
- `Alt`(`Option`) + `UP`(`DOWN`)

### 标签页间相互跳转 ###
- `Alt`(`Option`) + `LEFT`(`RIGHT`)

### 重写方法 ###
- `Ctrl`(`Command`) + `O`

### 代码上下移动 ###
- `Alt`(`Option`) + `Shift` + `UP`(`DOWN`)</br>
![](http://i.imgur.com/GQxf0E9.gif)

### 打开设置对话框 ###
- `Ctrl`(`Command`) + `Alt`(`Option`) + `S`

### 打开当前项目/模块属性 ###
- `Ctrl`(`Command`) + `Shift` + `Alt`(`Option`) + `S`

### 多行编辑 ###
- `Alt` + 鼠标滑选</br>
![](http://i.imgur.com/FLw7rxu.gif)
