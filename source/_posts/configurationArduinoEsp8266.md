---
title: 为ESP8266安装与配置ArduinoIDE
date: 2021-09-06 16:41:40
tags:
    - 教程
---
# Arduino IDE 安装 与 配置ESP8266开发环境
-----
> 笔者因为正在开发的一个项目需要用到ArduinoIDE,所以便顺便写下一个文章记录安装和配置过程。
> 

-----


## ArduinoIDE下载与安装
官方网站软件下载地址:
https://www.arduino.cc/en/software
![选择下载](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906121117.png)
选择适合自己的版本下载,本文章以windows安装版本示例


-----


![选项](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906134147.png)
进入安装界面,同意协议,选择路径,选项默认下一步即可


-----


![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906134337.png)
安装过程中会弹出提示安装设备驱动,都点击安装即可

提示安装完成后即完成安装


-----


## 为esp8266配置
启动Arduino IDE

弹出主界面后点击左上角的文件-首选项
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906161917.png)



-----


如图在箭头指向的框内输入
```
http://arduino.esp8266.com/stable/package_esp8266com_index.json
```
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906162134.png)


-----
可以在 网络 分页设置代理。因为服务器在国外,国内网络不一定能连接的上,推荐设置代理
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906162557.png)
完成后点“好”保存


-----
打开 开发板管理器
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906163130.png)
搜索esp8266 找到对应的平台版本 安装
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906163326.png)
等待下方的进度条读完即安装完成,如果超时或者安装失败请检查网络


-----
然后选择你对应的esp8266开发板即可
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906163541.png)


-----
在如同红框中配置波特率等设置
![](https://cdn.jsdelivr.net/gh/HK560/MyPicHub@master/res/pic/20210906163812.png)


-----
配置结束

