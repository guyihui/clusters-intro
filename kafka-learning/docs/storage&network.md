# 存储与网络性能测试报告
## 存储测试
### 一.计算机硬件简介
1. 使用get-physicaldisk命令获取基本硬盘信息  
![avatar](https://github.com/dnzlike/performance-test/blob/master/screenshots/get-physicaldisk.png?raw=true)
2. C盘、D盘各自使用情况  
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/OS(C).png?raw=true"
    width=40%>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/DATA(D).png?raw=true"
    width=40%>
</center>   

### 二.测试工具

1. 选用windows系统自带的性能监视器
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/%E6%80%A7%E8%83%BD%E7%9B%91%E8%A7%86%E5%99%A8.jpg?raw=true"
    width=80%>
</center>   

2. 创建新的数据收集器，选择监视PhysicalDisk，勾选D盘和C盘
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/PhysicalDisk.png?raw=true"
    width=80%>
</center>   

3. 选择输出文件格式以及监测时间间隔   
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/DataCollector.png?raw=true"
    width=80%>
</center> 

### 三.实验内容   

分别向C盘（SSD）和D盘（HDD）写入相同大小的数据，观察写入速率   
首先检测未执行操作前各自磁盘每秒Read/Write数据大小   
左C盘，右D盘   
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/C-before.jpg?raw=true"
    width=40%>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/D-before.jpg?raw=true"
    width=40%>
</center> 

测试用数据大小
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/TestFile.jpg?raw=true"
    width=40%>
</center>

分别将测试数据写入C盘和D盘   
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/copying.jpg?raw=true"
    width=40%>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/copying-C-D.jpg?raw=true"
    width=40%>
</center>

写入时每秒Read/Write数据大小   
左C盘，右D盘   

<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/C-after.jpg?raw=true"
    width=40%>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/D-after.jpg?raw=true"
    width=40%>
</center>   

### 实验结果及分析
可以看到，C盘在未执行操作时本身就具有较大的每秒传输数据量。可能是因为C盘作为系统盘，一直在运转着为计算机做很多事情，而D盘在未进行操作时就几乎没有运转。在传输数据时两者表现也相差巨大，C盘传输速度是D盘的近千倍左右，为了确保没有操作失误还多实验了几次都是这样的结果。一方面原因可能是C盘作为SSD，传输速率确实是要比HDD快上不少，另一方面猜测从C盘向D盘传输时可能并没有发生实际上的全部写入，否则无法解释速率慢了一千倍却还能在差不多的时间内完成。

## 网络测试

### 一.测试工具
iperf
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/iperf.jpg?raw=true"
    width=80%>
</center>

### 二.测试内容
TCP测试   
服务器：
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/TCP-Server.jpg?raw=true"
    width=80%>
</center>

客户端：
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/TCP-Client.jpg?raw=true"
    width=80%>
</center>

UDP测试  
服务器：
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/UDP-Server.jpg?raw=true"
    width=80%>
</center>

客户端：
<center>
    <img src="https://github.com/dnzlike/performance-test/blob/master/screenshots/UDP-Client.jpg?raw=true"
    width=80%>
</center>