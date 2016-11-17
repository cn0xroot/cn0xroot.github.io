#Exploring the Wireless World with USRP |Aircraft Tracking with ADS-B

**温馨提示：请自觉遵守无线电管理法规，依法设置和使用无线电设备**

Author:[雪碧0xroot](http://www.spriteking.com)@漏洞盒子安全团队 </span>[cn0xroot.github.io](http://www.0xroot.cn)

0x00 前言
---------

**USRP是数款流行的SDR硬件中功能和应用都相对成熟的一款产品，从WIFI协议、ZigBee协议、RFID协议、GSM通信系统、LTE
4G通信系统到飞机通信、卫星通信USRP都能很好的进行支持。软件开发工程师可以用它开发应用，安全工程师则用它来测试、研究相关的无线通信协议。**

很大一部分玩过电视棒的小伙伴都使用过电视棒+dump1090的方案实现过追踪飞机飞行轨迹这一功能。之所以能够很容易的跟踪飞机,是因为航空CNS(通信导航监视)系统里大量采用非常古老的无线标准。

![dump1090](http://image.3001.net/images/20161115/1479189205495.png!small)

（电视棒+dump1090 2D）

![Clipboard
Image.png](http://image.3001.net/images/20161115/14791886547297.png!small)

（图片来源：[http://slideplayer.com/slide/2547225](http://slideplayer.com/slide/2547225/)）

二次监视雷达(SSR)系统,地面站发射1030MHz的查询信号,飞机接收到此信号之后在1090MHz发射应答信号,信号中包含了飞机的一些信息,显示在空管的雷达屏幕上。还有空中防撞系统(TCAS),飞机可以自己发射1030MHz的查询信号,其他飞机接收到此信号之后在1090MHz发射应答信号,因此一架飞机得以"看到"周围的飞机。由于以上的查询-应答模式在飞机很多的时候显得效率不是那么高,因此新出现了一种ADS-B方式。在ADS-B中,每架飞机不等查询,主动广播自己的信息,这时监视和防撞需要做的就仅仅是接收了。

在通用航空当中ADS-B信号经常在978MHz发射，在商业飞行中ADS-B信号经常在1090MHz发射。

0x01 HardWare
-------------

PC：Ubuntu OR Mac

SDR：USRP、天线、USB数据线

![USRP](http://image.3001.net/2016/11/bd0dd8ac5f1e795033649a42110072e6.jpeg)

0x02 Software
-------------

### 2.1安装pip、pybombs

    apt-get update
    apt-get install git
    apt-get install python-pip

    pip install --upgrade pip
    pip install git+https://github.com/gnuradio/pybombs.git

    pybombs recipes add gr-recipes git+https://github.com/gnuradio/gr-recipes.git pybombs recipes add gr-etcetera git+https://github.com/gnuradio/gr-etcetera.git
    pybombs prefix init /usr/local -a myprefix -R gnuradio-default
    pybombs install gqrx gr-osmosdr uhd

以上内容是Ubuntu下安装SDR相关软件的方法，在Mac OSX中则可以使用[mac
port](https://www.macports.org/install.php) 进行安装。

### 2.2下载USRP镜像\

使用pybombs安装完UHD（USRP Hardware
Driver）后还需下载固件镜像以及FPGA镜像，执行：

    python  /usr/local/lib/uhd/utils/uhd_images_downloader.py

### ![Image](http://image.3001.net/images/20161114/1479113902337.png!small)
插入USRP后可执行：

    uhd_find_devices

    或者

    uhd_usrp_probe

来查看设备信息：

### ![Clipboard Image.png](http://image.3001.net/images/20161115/14791897937569.png!small)
### 2.3编译安装gr-air-modes:

    git clone https://github.com/bistromath/gr-air-modes
    cd gr-air-modes
    mkdir build
    cd build
    cmake ..
    make
    sudo make install
    sudo ldconfig

### 2.4 安装谷歌地球

**Ubuntu 32 bit:**

    wget http://dl.google.com/dl/earth/client/current/google-earth-stable_current_i386.deb

**Ubuntu 64 bit:**

    wget http://dl.google.com/dl/earth/client/current/google-earth-stable_current_amd64.deb

    sudo dpkg -i google-earth-stable_current_amd64.deb

**Mac osx：**

    wget https://dl.google.com/earth/client/advanced/current/GoogleEarthMac-Intel.dmg

![Clipboard
Image.png](http://image.3001.net/images/20161115/14791890906731.png!small)

0x03 解码飞机信号&导入谷歌Earth
-------------------------------

    cd gr-air-modes/apps/
    ./modex_rx -K test.kml 

执行App目录下的modex\_rx，开始接收并解码来自飞机的1090MHz无线信号，-K参数即把解码到的航班号、经纬度、飞行速度等等保存为.kml文件。

打开谷歌地球：添加--&gt;网络链接--&gt;

![Clipboard
Image.png](http://image.3001.net/images/20161115/14791907352309.png!small)

新建链接名称，以及kml文件的绝对路径：

![Clipboard
Image.png](http://image.3001.net/images/20161115/14791908433470.png!small)

设定刷新时间，以及是否在刷新时前往该视图：

![Clipboard
Image.png](http://image.3001.net/images/20161115/14791909528504.png!small)

如果开启刷新前往视图后，google-earth会自动定位到你所在的区域并显示接收到区域上空的飞机。

![Clipboard
Image.png](http://image.3001.net/images/20161115/14791912477419.png!small)

(飞机飞行轨迹 3D)

地图上显示飞机的航班号，双击飞机图标可以显示该飞机高度、飞行速度等信息。 

0x04 demo视频
-------------
[v.qq.com](https://v.qq.com/x/page/e0346ll12xf.html)


0x05 refer
----------

<https://kb.ettus.com/Implementation_of_an_ADS-B/Mode-S_Receiver_in_GNU_Radio>

<http://www.freebuf.com/articles/wireless/77819.html>

[USRP B200: Exploring the Wireless
World](https://www.ettus.com/product/details/UB200-KIT)

[Aircraft Tracking with Mode S: Modez & Aviation
Mapper](http://spench.net/drupal/research/mode-s)

<https://media.blackhat.com/bh-us-12/Briefings/Costin/BH_US_12_Costin_Ghosts_In_Air_WP.pdf>

<https://www.rs-online.com/designspark/10-things-you-can-do-with-software-defined-radio>

