---
layout:     post
title:      树莓派环境配置
subtitle:   树莓派入门系列03
date:       2018-10-30
author:     YB
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Raspberry
    - 树莓派
---

# 前言
上一篇树莓派已经安装好系统，接下来安装OpenCV
[所用到软件百度云备份](https://pan.baidu.com/s/1ha2II2KaIqmItN3fUXtAqA)
提取码：61qz

# 安装教程
安装OpenCV3.4.3
[官方教程](https://docs.opencv.org/3.4.3/d2/de6/tutorial_py_setup_in_ubuntu.html)
[参考教程1](https://blog.csdn.net/cocoaqin/article/details/78163171)
[参考教程2](https://blog.csdn.net/ksws0292756/article/details/79511170)

# 安装步骤

## 安装依赖库和cmake
sudo apt-get install cmake  
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev libavformat-dev libjpeg.dev libtiff4.dev libswscale-dev libjasper-dev

```
[compiler] sudo apt-get install build-essential
[required] sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
[optional] sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```


## 下载opencv包
有两种方式：
### 方式1：下载release
[源码下载地址](https://opencv.org/releases.html) 选择sources 
`树莓派git有点慢 windows科学桑网 下载完再用pscp传到树莓派`
#### pscp
从windows传文件到树莓派,和scp差不多

用法：
1. 打开cmd,切换到putty所在目录
![](_v_images/20190416122426732_141.png)

2. 传文件到树莓派
`pscp F:\opencv-4.1.0.zip pi@192.168.0.105:/home/pi/opt/`

![](_v_images/20190416122822724_28096.png)
`这个目录很关键，开始少了/pi/ 一直报错`

3. 解压
`unzip opencv-4.1.0.zip`
![](_v_images/20190416123131898_10032.png)

### 方式2：从git仓库里下载
```
sudo apt-get install git
git clone https://github.com/opencv/opencv.git
```


## 创建编译文件夹
cd opencv
mkdir build
cd build

##  cmake
cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local ..

## 编译
sudo make
`很慢很慢`

## 安装
sudo make install

## 添加路径
### 1执行
sudo vi /etc/ld.so.conf.d/opencv.conf 

### 2打开的是一个空白文件，文件末尾添加
/usr/local/lib  

### 3使生效
sudo ldconfig

## .配置bash

### 1执行
sudo vi /etc/bash.bashrc  

### 2文件末尾添加
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig  
export PKG_CONFIG_PATH  

### 3使生效
source /etc/bash.bashrc

### 更新
sudo updatedb  

## 测试
### 查看版本
pkg-config --modversion opencv

### 官方例子
cd 到 opencv-3.4.3/samples/cpp/example_cmake
cmake .
make
./opencv_example

可以看到摄像头打开