[参考链接](https://www.pyimagesearch.com/2018/09/26/install-opencv-4-on-your-raspberry-pi/)


## Step #1: Expand filesystem 
- `sudo raspi-config`
- 选择 “Advanced Options” ，然后选择“Expand filesystem”选择`<Finish>` 后重启
- 使用`df -h`查看效果：
![](_v_images/20190416210014920_8541.png)
>没用的软件LibreOffice and Wolfram可以删掉`尚未测试`
```
sudo apt-get purge wolfram-engine
sudo apt-get purge libreoffice*
sudo apt-get clean
sudo apt-get autoremove
```

## Step #2: 安装opencv4 依赖
`sudo apt-get update && sudo apt-get upgrade`
`sudo apt-get install build-essential cmake unzip pkg-config`

- 图像视频处理的常用库
```
sudo apt-get install libjpeg-dev libpng-dev libtiff-dev
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
```

- 安装GTK，GUI后端
`sudo apt-get install libgtk-3-dev`

-  install a package which may reduce pesky GTK warnings

`sudo apt-get install libcanberra-gtk*`

- installing two packages which contain numerical optimizations for OpenCV
`sudo apt-get install libatlas-base-dev gfortran`

- install the Python 3 development headers
`sudo apt-get install python3-dev`

## Step #3: 下载安装包
```
cd ~/opt
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.1.0.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.1.0.zip
```

## Step #4: 配置Opencv4 用到的python3虚拟环境
### 安装pip
```
sudo apt-get install python-pip  
# 原文这个链接上不去↓
# wget https://bootstrap.pypa.io/get-pip.py
# sudo python3 get-pip.py
```


### 虚拟环境
虚拟环境是一种特殊的工具，通过为每个环境创建独立的，独立的 Python环境来保持不同项目所需的依赖关系

#### 虚拟环境用到的两个软件virtualenv和 virtualenvwrapper
这里和原文不太一样,原文执行`source ~/.profile`后报错:
> source ~/.profile
>/usr/bin/python3.5: Error while finding module specification for 'virtualenvwrapper.hook_loader' (ImportError: No module named 'virtualenvwrapper')
>virtualenvwrapper.sh: There was a problem running the initialization hooks.

`sudo apt-get install python3-pip`
`sudo pip3 install virtualenv virtualenvwrapper`

```
# 原文方法
sudo pip install virtualenv virtualenvwrapper
sudo rm -rf ~/get-pip.py ~/.cache/pip
```

#### 更新profile文件
###### 方式1
- 打开文件
`vi ~/.profile`

- 加上如下内容
```
# virtualenv and virtualenvwrapper
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source /usr/local/bin/virtualenvwrapper.sh
```

##### 方式2
直接在bash输入:
```
echo -e "\n# virtualenv and virtualenvwrapper" >> ~/.profile
echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.profile
echo "export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3" >> ~/.profile
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.profile
```

##### 更新
source ~/.profile

#### 创建虚拟环境
- 创建一个python3的虚拟环境,名字是cv
`mkvirtualenv cv -p python3`

- 检查
输入`workon cv`
![](_v_images/20190416221349161_25390.png)

#### 安装numpy
`pip install numpy`

## Step #5 CMake & 编译
### build目录
```
cd ~/opt/opencv-4.1.0
mkdir build
cd build
```

### cmake
其中的`OPENCV_EXTRA_MODULES_PATH`路径根据自己实际情况修改!
```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D OPENCV_EXTRA_MODULES_PATH=~/opt/opencv_contrib-4.1.0/modules \
    -D ENABLE_NEON=ON \
    -D ENABLE_VFPV3=ON \
    -D BUILD_TESTS=OFF \
    -D OPENCV_ENABLE_NONFREE=ON \
    -D INSTALL_PYTHON_EXAMPLES=OFF \
    -D BUILD_EXAMPLES=OFF ..
```

#### 检查以下两处
![Ensure that “Non-free algorithms” is set to “YES”. This will allow you to use patented algorithms such as SIFT/SURF for educational purposes.](_v_images/20190417113130662_25445.png)

![The CMake command allows us to generate build files for compiling OpenCV 4 on the Raspberry Pi. Since we’re using virtual environments, you should inspect the output to make sure that the compile will use the proper interpreter and NumPy.](_v_images/20190417113242321_916.png)


### 建议增加swap
建议增加交换空间,这样可以使用树莓派的四核进行编译,而不会由于内存耗尽导致编译挂起.

- 打开
`sudo nano /etc/dphys-swapfile`

- 修改
```
# set size to absolute value, leaving empty (default) then uses computed value
#   you most likely don't want this, unless you have an special disk situation
# CONF_SWAPSIZE=100
CONF_SWAPSIZE=2048
```
- 保存退出
保存所做的修改，按下Ctrl+O。想要退出，按下Ctrl+X

- 重启交换服务
```
sudo /etc/init.d/dphys-swapfile stop
sudo /etc/init.d/dphys-swapfile start
```

注意:增加交换空间会降低内存卡寿命,所以我们只在编译时短暂的用一会. 注意：请在安装OpenCV + Python后备份.img文件

### 编译
`make -j4`
使用4核进行编译,如果出错,请去掉`-j4`再试

### 安装OpenCV4
```
sudo make install
sudo ldconfig
```

- 编译完成后把`CONF_SWAPSIZE`改回100MB

## Step #6 把OpenCV4链接到Python虚拟环境
```
cd ~/.virtualenvs/cv/lib/python3.5/site-packages/
ln -s /usr/local/python/cv2/python-3.5/cv2.cpython-35m-arm-linux-gnueabihf.so cv2.so
cd ~
```

其中第二行根据自己实际位置，我用的是：
`ln -s /usr/local/lib/python3.5/site-packages/cv2/python-3.5/cv2.cpython-35m-arm-linux-gnueabihf.so cv2.so`
位置是通过`find -name *cv2.cpython-35m-arm-linux-gnueabihf.so*`搜出来的

## Step #7 测试OpenCV4
```
workon cv
python
>>> import cv2
>>> cv2.__version__
'4.1.0'
>>> exit()
```