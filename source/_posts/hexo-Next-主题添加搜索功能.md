---
title: hexo - Next 主题添加搜索功能
date: 2019-03-10 16:13:32
categories:
	- hexo
tags:
	- hexo
	- nexT
---

参考自[Yaya's blog](https://yashuning.github.io/2018/06/29/hexo-Next-%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%90%9C%E7%B4%A2%E5%8A%9F%E8%83%BD/)添加Hexo搜索功能

## 安装相关插件

首先安装搜索插件： `hexo-generator-searchdb`

在hexo路径文件夹下执行：

```
$ npm install hexo-generator-searchdb --save
```

<!--more-->

## 配置hexo

安装完成，在hexo的配置文件`_config.yml`中添加：

```
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```

## 配置主题

Next 主题自带搜索设置，编辑主题配置文件：`_config.yml`

找到文件中 Local search 的相关配置，设为 `true`

```
# Local search
local_search:
  enable: true
```

## hexo 重新部署

```
$ hexo d -g
```