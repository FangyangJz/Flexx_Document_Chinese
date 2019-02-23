# 安装说明

## 安装
> pip install flexx

> conda install flexx -c conda-forge

使用 pip 或者 conda 方式安装, 依赖包会自动安装

## 依赖包

作为纯 Python 和跨平台项目, Flexx (几乎) 可以在任何 [Python + 浏览器] 的环境下运行.   
Flexx 专门为 Python 3.5+ 编写, 也可以运行在 Pypy 上面.

Flexx 还依赖以下包 ( 所有需要纯 Python 支持, 后面3个项目给 flexxui 提供支持 ) :
* [Tornado](https://tornado.readthedocs.io/en/stable/)
* [PScript](http://pscript.readthedocs.io/)
* [Webruntime](http://webruntime.readthedocs.io/)
* [Dialite](http://dialite.readthedocs.io/)

## 浏览器支持

Flexx 立志于支持全部现代的浏览器, 包括 Firefox, Chrome 和 Edge.  
兼容 IE10 及以上版本, 但是可能会有些小问题.

运行桌面 apps, 我们推荐事先安装 Firefox 或者 nw.js.

## 目前状态

从 0.5 版本 (2018年9月) 开始, Flexx 处于测试状态;  
虽然部分 API 可能已经改变, 但是我们依然重视后向兼容.