#如何用极路由+OpenWrt+RTL电视棒搭建一台SDR服务器
#How To Building a SDR Server With OpenWrt And RTL 

##0x00 前言
近期因为有个从异地捕获无线信号的需求，便尝试着用OpenWrt+公网IP搭建了一台SDR服务器。如果有小伙伴嫌SDR硬件天线看起来太乱、或者电脑没有足够的USB接口也可在局域网搭建SDR服务器通过TCP/IP调用SDR硬件。
![](http://image.3001.net/images/20161205/14809227662572.jpg)
##0x01 获取root
刚买的极路由关闭了root功能，需要开启路由的开发者模式后才能通过SSH连入shell交互界面。
申请开发者模式流程：进入路由器后台-云平台-路由器信息-高级设置-申请-绑定手机-输入验证码-绑定微信-微信账号绑定极路由账号。

下图是开启开发者模式前后的Nmap扫描结果：

![Nmap](http://image.3001.net/images/20161205/14809086971604.png!small)

开启开发者模式后可通过1022端口进入路由器shell界面：

```
ssh root@192.168.199.1 -p 1022
```

![ssh](http://image.3001.net/images/20161205/14809087239156.png!small)

##0x02 极路由刷不死uboot
开启开发者模式后可对设备进行刷机，为了防止设备变砖可在设备刷入具有不死uboot之称的Breed Bootloader。
在 http://breed.hackpascal.net/ 页面找到对应型号的uboot （极路由1s：HC5661、极路由2s：HC5761、极路由3：HC5861）

![](http://image.3001.net/images/20161205/14809087398580.png!small)
###下载、刷入uboot
<pre>
cd /tmp
wget http://breed.hackpascal.net/breed-mt7620-hiwifi-hc5861.bin
mtd -r write  breed-mt7620-hiwifi-hc5861.bin
</pre>

![](http://image.3001.net/images/20161205/1480908746663.png!small)

显示rebooting后等待路由重启完成。

重启完毕后三灯亮起，这时需断开电源，按住路由器的RST重置键然后再通电，当看到电源灯闪烁时可以松开RST键。电脑通过网线接入后自动获取ip，用浏览器192.168.1.1即可登陆Breed控制台。

![](http://image.3001.net/images/20161205/14809087658372.png!small)

安全起见，备份所有内容：

![](http://image.3001.net/images/20161205/14809087721653.png!small)

![](http://image.3001.net/images/20161205/14809194869145.png!small)

##0x03 极路由刷OpenWrt

由于SDR服务器需要一个USB接口来插电视棒，所以需要在购买极路由的时候选一款带USB接口的机器。其它带USB接口的OpenWrt路由器也适用下文的内容.

###查看CPU信息：
```
cat /proc/cpuinfo
```

![](http://image.3001.net/images/20161205/14809196479728.png!small)

### [下载OpenWrt固件](http://rssn.cn/roms/)：选择自己路由器对应的版本
<pre>
cd /tmp
wget http://rssn.cn/roms/openwrt-15.05-ramips-mt7620-hc5861-squashfs-sysupgrade.bin
sysupgrade -F -n openwrt-15.05-ramips-mt7620-hc5861-squashfs-sysupgrade.bin
</pre>

![](http://image.3001.net/images/20161205/14809200421471.png!small)

##0x04OpenWrt安装RTL驱动
OpenWrt刷入重启后，进入管理界面：

http://192.168.1.1

user:root

pass：root

###设置SSH密码

![](http://image.3001.net/images/20161205/14809202608798.png!small)

<pre>
ssh root@192.168.1.1
</pre>

![](http://image.3001.net/images/20161205/1480920352518.png!small)

Openwrt可以使用opkg命令对软件包进行管理

<pre>
opkg update
opkg list |grep rtl
opkg install rtl-sdr
</pre>

![](http://image.3001.net/images/20161205/14809204315790.png!small)

安装完成后便可将电视棒插入路由器的USB接口:
![](http://image.3001.net/images/20161205/14809257263308.jpg)

###启动OpenW上的rtl-sdr
OpenWrt终端执行：
```
rtl_tcp -a 192.168.1.1 -n 8 -b 8
```
之后OpenWrt上将开启1234端口：

![](http://image.3001.net/images/20161205/1480920925328.png!small)

###0x05使用SDR服务
客户机上执行：
```
osmocom_fft -W -s 2000000 -f 144000000 -a 'rtl_tcp=192.168.1.1:1234'
```

![](http://image.3001.net/images/20161205/14809209723917.png!small)

```
osmocom_fft -F -s 1.5e6 -f 101e6 -a 'rtl_tcp=192.168.1.1:1234'
```
![](http://image.3001.net/images/20161205/14809254388370.png)

#### gqrx
![](http://image.3001.net/images/20161205/14809217008674.png)

###0x06利用场景

1.可在机场塔台、港口等地方使用SDR服务器监测ADB-S、AIS（船舶自动识别系统Automatic Identification System）

2.利用SDR+WIFI窃取 语音、图像数据：
![](http://image.3001.net/images/20161205/14809233561479.png)
![](http://image.3001.net/images/20161205/14809247082445.png)
更多细节可参考DefCon Paper:
![](http://image.3001.net/images/20161205/1480924553194.png)
![](http://image.3001.net/images/20161205/14809245893246.png)
![](http://image.3001.net/images/20161205/14809246382759.png)

[How Hackers Could Wirelessly Bug Your Office](https://www.blackhat.com/docs/us-15/materials/us-15-Cui-Emanate-Like-A-Boss-Generalized-Covert-Data-Exfiltration-With-Funtenna.pdf)

[Video:YouTuBe](https://www.youtube.com/watch?v=5GnMj5cus4A&feature=youtu.be)

MayBe还能通过SDR服务器利用MouseJack漏洞对办公区域的键盘鼠标输入进行监听：
[http://www.freebuf.com/articles/terminal/97011.html](http://www.freebuf.com/articles/terminal/97011.html)

###0x07 Refer

https://github.com/rssnsj/openwrt-hc5x61

http://www.binss.me/blog/install-openwrt-on-hiwifi-router/

http://www.right.com.cn/forum/thread-161906-1-1.html

http://www.levey.cn/352.html

http://www.right.com.cn/forum/thread-161906-1-1.html

http://yo2ldk.blogspot.com/2016/03/wireless-sdr-receiver.html

http://adventurist.me/posts/0050

http://sdr.osmocom.org/trac/wiki/rtl-sdr
