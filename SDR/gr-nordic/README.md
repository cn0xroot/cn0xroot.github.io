###0x01 系统安装
下载[Ubuntu 16.04](http://www.ubuntu.com/download/desktop)

###0x02 搭建SDR开发环境
安装pip、pybombs
<pre>
apt-get update
apt-get install git
apt-get install python-pip
pip install --upgrade pip
pip install git+https://github.com/gnuradio/pybombs.git
</pre>
获取GnuRadio的安装库
<pre>
pybombs recipes add gr-recipes git+https://github.com/gnuradio/gr-recipes.git  
pybombs recipes add gr-etcetera git+https://github.com/gnuradio/gr-etcetera.git
</pre>
安装SDR常用软件
<pre>
pybombs install osmo-sdr rtl-sdr gnuradio hackrf airspy gr-iqbal libosmo-dsp gr-osmosdr gqrx 
</pre>


使用pybombs安装bladeRF会报错，这里选择源码编译：
<pre>
git clone https://github.com/Nuand/bladeRF
cd bladeRF/host
mkdir build
cd build
cmake ../
make
sudo make install
sudo ldconfig
</pre>

###0x03 编译gr-nordic
<pre>
git clone https://github.com/BastilleResearch/gr-nordic/
cd gr-nordic/
mkdir build
cd build/
cmake ../
make
sudo make install
sudo ldconfig
</pre>

###0x04 安装WireShark
<pre>apt-get install wireshark</pre>

Ubuntu系统中，访问网络端口需要root权限，而wireshark的只是/usr/share/dumpcap的一个UI，/usr/share/dumpcap需要root权限，所以没法non-root用户无法读取网卡列表。解决办法使用sudo wireshark启动抓包，但使用root权限启动wireshark就不能使用lua脚本：
解决方案：
<pre>
sudo -s  
groupadd wireshark  
usermod -a -G wireshark $你的用户名  
chgrp wireshark /usr/bin/dumpcap  
chmod 750 /usr/bin/dumpcap 

setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap
getcap /usr/bin/dumpcap
</pre>
当输出为：
<pre>
/usr/bin/dumpcap = cap_net_admin,cap_net_raw+eip 
</pre>

![](http://image.3001.net/images/20160927/14749447459478.png)

即为设置生效。注销登录状态或者重启系统启用配置。

###0x05 数据包捕获

![](http://image.3001.net/images/20160927/14749450226613.png)

gr-nordic项目中include里边包含了nordic的tx、rx、API头文件，lib文件夹则是该项目依赖的一些库文件，example文件包含了Microsoft鼠标以及扫描、嗅探使用Nordic北欧芯片键鼠的利用脚本，wireshark文件夹中则是对扫描、嗅探到的数据包进行分析所需的lua脚本。
<pre>
gr-nordic$ wireshark -X lua_script:wireshark/nordic_dissector.lua -i lo -k -f udp
</pre>
<pre>
gr-nordic$cd example
gr-nordic/example$./nordic_sniffer_scanner.py
</pre>

![](http://image.3001.net/images/20160926/14748794773330.png)
###0x06 演示视频
[Video:v.qq.com](http://v.qq.com/x/page/x0322x7vcm4.html)

[Video:YouTuBe](https://www.youtube.com/watch?v=EsZNfhmIu64&feature=youtu.be)


### 0x07 Thanks & Refer
[gr-nordic: GNU Radio module and Wireshark dissector for the Nordic Semiconductor nRF24L Enhanced Shockburst protocol. ](https://github.com/BastilleResearch/gr-nordic/)

[孤独小白：GNU Radio教程（一）](http://www.white-alone.com/GNURadio%E6%95%99%E7%A8%8B_1/)

[Sniffing with Wireshark as a Non-Root User](http://packetlife.net/blog/2010/mar/19/sniffing-wireshark-non-root-user/)

[Bastille 巴士底狱](https://twitter.com/bastillenet)安全研究员：[Marc Newlin](https://twitter.com/marcnewlin)
、[Balint Seeber](https://twitter.com/spenchdotnet)
####Author：[雪碧0xroot](http://www.0xroot.cn) @[漏洞盒子安全团队 VULBOX Security Team](https://www.vulbox.com/)  



