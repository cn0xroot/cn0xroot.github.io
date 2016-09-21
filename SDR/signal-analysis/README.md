#0×00 前言

![](http://image.3001.net/images/20160815/14712438691914.jpg!small)

前段时间在[《HackRF入门：家用无线门铃信号重放》](http://www.freebuf.com/news/topnews/83650.html) 一文中通过HackRF录制、重放了无线遥控信号，不过一直没来得及对信号进行分析，刚好在国外网站看到有大牛对遥控信号进行了分析（详见refer部分）。在这里便按照国外大牛分析无线遥控信号的方法来依葫芦画瓢。

>*本文仅分享信号分析方式，因信号调制编码方式有所不同，如数据分析有出错，希望大家不要打我=￣ω￣= 摸摸大

#0×01 环境搭建

Mac可使用port(/www.macports.org) 或者brew(brew.sh)安装GnuRadio依赖套件:
 <pre> 
sudo port install gnuradio 
sudo port install hackrf
sudo port install rtl-sdr
sudo port install gr-osmosdr gqrx
sudo port install hackrf
</pre> 

完成上面的工作后便能在Mac环境中使用电视棒、HackRF、GnuRadio了。    

#0×02 Recording 信号录制

录制遥控信号的方式有很多，如电视棒+SDR-sharp录制wav音频格式数据、通过HackRF命令终端录制RAW格式数据，本文使用GNURadio+SDR硬件（rtl-sdr、HackRF、BladeRF等）来实现这一功能：

![](http://image.3001.net/images/20160812/14709826287531.png)

左侧RTL-SDR Source将使用SDR硬件接收315MHz无线信号,采样率为2M，右上WX GUI Waterfall sink将接收到的信号通过瀑布图在PC上显示捕获的无线信号，右下角File Sink将捕获到的无线数据包储存到/tmp/test.cfile文件中。执行流图并摁下遥控可看到如下效果图：

![](http://image.3001.net/images/20160812/1470982909985.png)

个人比较喜欢使用gr-fosphor的瀑布图模块来对捕获到的信号在瀑布图上进行展示：

![](http://image.3001.net/images/20160812/14709976134247.png) 

结束GnuRadio流图后，查看/tmp目录下的test.cfile:

![](http://image.3001.net/images/20160812/14709829982492.png)  

#0×03 Analysis 信号分析

分析信号可使用音频处理软件Audacity：
![](http://image.3001.net/images/20151121/14481041363948.png)
    

不过这种方式需要肉眼将波形转化成0跟1，看起来比较容易眼花。maybe，只有老司机才能很快很准确地用这种方式完成分析任务。

##3.1 安装inspectrum

在这篇文章中我们将通过[inspectrum](https://github.com/miek/inspectrum)
这个工具来分析信号，配合Python将信号转成二进制数据。
<pre> 
sudo port install fftw-3-single cmake pkgconfig qt5
git clone https://github.com/miek/inspectrum.git
mkdir build
cd build
cmake ..
make
sudo make install
</pre> 


</pre> 
<pre> 
inspectrum -h
Usage: inspectrum [options] file
spectrum viewer

Options:
  -h, --help       Displays this help.
  -r, --rate <Hz>  Set sample rate.

Arguments:
  file             File to view. 
</pre> 

##3.2 数据导入、分析


`inspectrum /tmp/test.cfile 
`

![](http://image.3001.net/images/20160812/14709855392599.png!small)  

通过左侧Spectrogram参数的调节、缩放工具，我们可以实现波形图的放大缩小，颜色深浅调节：

![](http://image.3001.net/images/20160812/14709855687940.png)   

下方Time selection可对波形进行划分：    

![](http://image.3001.net/images/20160812/14709856018893.png)   

对Symbols进行递增，直至囊括一个信号波形区域：

![](http://image.3001.net/images/20160812/14709856208390.png)

右键—>Add derved plot—>Add amplitude plot:    

![](http://image.3001.net/images/20160812/1470985643476.png) 

效果如下：    

![](http://image.3001.net/images/20160812/14709856713054.png)  

对部分参数进行微调：    

![](http://image.3001.net/images/20160812/14709857405198.png)

![](http://image.3001.net/images/20160812/14709857405198.png)

导出波形数据：

![](http://image.3001.net/images/20160812/1470985854699.png)

此时在终端获得波形宽度数据：    

![](http://image.3001.net/images/20160812/14709858864572.png)

##3.3 解码

接下来我们可通过Python将这些数据转成0、1,，test.py代码如下：（if i > x  x的值根据自身实际情况决定，建议取最大值跟最小值区间的自然数）
<pre>

s = ''
a = [0.121182, 0.00224696, 0.00227361, 0.00222253, 0.121036, 0.121293, 0.12126, 0.00220722, 0.121013, 0.00221486, 0.00230146, 0.00230048, 0.120959, 0.120975, 0.12077, 0.00227199, 0.120701, 0.00226761, 0.00234306, 0.00225335, 0.120851, 0.120784, 0.12084, 0.00224014, 0.120892, 0.00221627, 0.00222881, 0.00219768, 0.121157, 0.00224349, 0.00221741, 0.00223827, 0.120798, 0.00237988, 0.00226093, 0.00232855, 0.120649, 0.120813, 0.121032, 0.00222553, 0.120876, 0.00221533, 0.00225347, 0.00228226, 0.120759, 0.120718, 0.12042, 0.00218557, 0.120344, 0.00222487, 0.00224753, 0.00227552, 0.120383, 0.120384, 0.120275, 0.00224362, 0.120611, 0.00219556, 0.00227022, 0.00224123, 0.120514, 0.120328, 0.12068, 0.0022916, 0.120735, 0.12043, 0.120697, 0.00224807, 0.120399, 0.120808, 0.120405, 0.00222214, 0.120512, 0.120833, 0.120495, 0.00226469, 0.120727, 0.120617, 0.120534, 0.00222499, 0.120441, 0.120626, 0.120297, 0.00208249, 0.120539, 0.120365, 0.120612, 0.00214876, 0.120545, 0.120262, 0.120739, 0.00228899, 0.12051, 0.120525, 0.120172, 0.00214644, 0.120678] 
for i in a:
	if i > 0.03:
		s +='1'
	else:
		s +='0'
print s	
</pre> 

![](http://image.3001.net/images/20160812/14709862223011.png)  

 

0×04 replay 信号重放

通过上述方式，我们已对SDR捕获到的无线信号进行分析，并把信号文件转换成了二进制数据，接下来可使用GnuRadio对数据进行重放、修改测试，或者使用RFcat+Python实现廉价的重放Hacking。

![](http://image.3001.net/images/20160812/14709993654189.jpg)        

#0×05 refer

[decoding-a-garage-door-opener-with-an-rtl-sdr-5a47292e2bda#.qu46ncrr3](https://medium.com/@eoindcoolest/decoding-a-garage-door-opener-with-an-rtl-sdr-5a47292e2bda#.qu46ncrr3)

[Mike Walters: Reversing digital signals with inspectrum – YouTube](https://www.youtube.com/watch?v=tGff31uGXQU)

[My quickest and easiest method for OOK signal decoding & replication in 2016 – YouTube](https://www.youtube.com/watch?v=1kFNMbdGb_4)

###*本文作者：[雪碧0xroot](http://www.0xroot.cn)@ 漏洞盒子安全团队，转载须注明来自FreeBuf.COM
