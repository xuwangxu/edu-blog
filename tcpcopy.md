# 分布式压力测试工具tcpcopy
## 1.tcpcopy是什么
 tcpcopy是一种请求复制（复制基于 TCP 的 packets）工具 ，通过复制在线数据包，修改 TCP/IP 头部信息，发送给测试服务器，可以把在线流量导入到测试系统中去。实时的模拟线上环境，达到在程序不上线的情况下实时承担线上流量的效果。

## 2.tcpcopy应用场景
 - 分布式压力测试：使用tcpcopy复制线上服务器的数据包，来进行压测服务。在高并发的场景下可能会出现bugs
 - 线上测试：验证线上新环境的系统性能，以及检验线上环境在高并发下可能会出现的问题
 - 回归测试
 - 性能比较

## 3.tcpcopy的工作原理
  - 如上图，tcpcopy包含了两部分：tcpcopy和intercept。tcpcopy运行在线上服务器(online server)，捕捉线上服务器的请求；intercept运行在辅助服务器（assiatant server）上，做一些辅助工作，例如响应tcpcopy的请求。应该指出的是，测试web服务应用应该在目标服务器上（target server）上进行。
 - tcpcopy默认利用原始套接字输入技术，在网络层捕捉数据包，进行必要的处理（包括tcp交互仿真网络延迟控制，共同上层交互仿真），在默认使用原始套接字输出技术，将数据包发送到目标服务器上（图中的粉色箭头）
 - 针对tcpcopy，目标服务器唯一要进行的操作是添加到辅助服务器的适合路由（图中绿色箭头的位置)。
 - 同样针对tcpcopy，intercept的作用是负责响应tcpcopy的对应请求。通过捕获响应数据包，intercept将提取响应头信息，并且通过特殊渠道（图中紫色箭头）发送响应头给tcpcopy。当tcpcopy接收到响应头的时候，它利用响应头信息来修改线上数据包的属性，从而继续发送另一个数据包。需要注意的是，从目标服务器到辅助服务器的路由可以看做一个黑洞。

##4.tcpcopy的安装部署
    tcpcopy代码下载地址：
    git clone  git://github.com/session-replay-tools/tcpcopy.git
    cd tcpcopy
    ./configure
    make
    make install
    程序安装目录: /usr/local/tcpcopy/
 
    注意：./configure 编译参数
    --offline           从pacp文件里面重播tcp数据流，用于离线模式
    --pcap-capture      在数据连接层捕获数据包
    --pcap-send         在数据链接层，而不是IP层发送数据包
    --with-pfring=PATH  设置PF_RING的库源代码路径
    --set-protocol-module=PATH  设置tcpcopy以额外的协议模块工作
    --single            如果intercept和tcpcopy都配置了--single选项，那么只有一个tcpcopy配合intercept工作
    --with-debug        将debug调试支持编译进tcpcopy（保存在一个日志文件）

##5.intercept安装部署
    intercept代码下载：
    git clone  git://github.com/session-replay-tools/intercept.git
    cd intercept
    ./configure
    make
    make install
    程序安装目录:/ usr/local/intercept
 
    注意：./configure 编译参数
    --single            如果intercept和tcpcopy都配置了--single选项，那么只有一个tcpcopy配合intercept工作
    --with-pfring=PATH  设置PF_RING的库源代码路径
    --with-debug        将debug调试支持编译进tcpcopy（保存在一个日志文件）

注意：

     安装时候intercept的时候出现报错：
     checking for pcap.h ... not found
 
    解决报错：
      yum -y install libpcap-devel

##6.intercept 和 tcpcopy命令
tcpcopy命令

    【参数详解】
     -x [transfer]:   定义server服务器的IP和端口，和目标服务器[测试服务器]的IP和端口。格式为：服务端IP:服务端port-目标IP:目标port ，服务端IP可以省略，也可以写成：服务端port-目标IP:目标port，IP地址和端口之间用”:”分割，server服务器的port与目标服务器的IP之间使用”-”分割。
    例如： tcpcopy –x 80-192.168.56.11:8080
    #<= 从server服务器的80端口复制请求到192.168.56.11的8080端口
 
    -H [ip_addr]:    将本地IP转换成指定IP
    -c <ip_addr,>:   更改源IP地址。从server服务器复制请求到测试服务器时，此时的客户端的IP地址改变成指定的IP地址之一。
    例如：
    tcpcopy –x 8080-192.168.56.11:8080 –c 192.168.56.x 
    #<= 将server服务器的8080端口的请求，复制到192.168.56.11的8080端口。并更改此时的来源IP地址为192.168.56.0/24中的一个。
 
    -n <num>：       复制过去的流量是当前server服务器流量的多少倍。最大值为1023.多重复制基于修改端口号，复制的倍数越大，端口冲突的几率就越大。尤其是源地址少，而且还是短连接很多的情况。复制流量的倍数越少，tcpcopy的效果会更好。
    例如：tcpcopy -x 80-192.168.0.2:8080 –n 3
    #<=从本地的80端口复制请求到192.168.0.2的8080端口，且复制的流量是当前流量的3倍
    -f <num>：       使用这个参数控制更改端口进程的数量，tcpcopy多实例运行的时候减少冲突的概率。多个tcpcopy实例之间 –f 的值应该不同，最大值不能超过1023.
 
    -s <server,> intercept 服务列表：
    格式：ip_addr1:port1, ip_addr2:port2, ...

intercept命令

    【参数详解】
     -i  <device,>     监听的设备名称。通常为驱动设备的名称，例如eth0
     -F <filter>       设置用户过滤器，例如pcap过滤器
     -n <num>          设置联合报的最大数量
     -p <num>          设置监听的tcp端口，默认值是36524
     -s <num>          设置拦截hash表的大小
     -l <file>         在文件里保存日志信息
     -P <file>         保存PID到指定文件里，仅仅配合-d参数的时候使用
     -b <ip_addr>      监听的设备名称
     -d                以守护进程方式运行

##7.运行tcpcopy： 离线模式
  tcpcopy以 ./configure  --offline方式编译， intercept 是以./configure方式编译. tcpcopy部署在目标服务器（target server）,intercept部署在辅助服务器(assistant server)
   需要三台服务器做演示：线上服务器（online server）、测试服务器【或者成为目标服务器】（target srver）、辅助服务器 (assistant server)

   （1） 线上服务器（online server 192.168.56.11）

     [root@linux-node1 local]# tcpdump -i eth0 tcp and port 8888 -s 0 -w /tmp/online.pcap  
     #<=抓取本地eth0端口,过滤类型为tcp的数据包，存储到/tmp/online.pcap 
     【选项参数】
     -i eth0: 抓取接口为eth0的数据包
     -s 0: 抓取数据包默认抓取长度为68字节，加上-s 0 后可以抓取完整的数据包
     -c 100: 只抓取100个数据包
     tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
     -w: 保存成/tmp/online.pcap 文件

（2）目标服务器（target server 192.168.56.53）

     [root@nfs-client2 sbin]# tcpcopy -i /tmp/online.pcap -x 8888-192.168.56.53:8080 -s 192.168.56.12 -c 192.168.50.x
     #<=tcpcopy将读取本地文件/tmp/online.pcap的8888端口的数据包，更改其客户端IP地址为192.168.50.0/24其中的一个地址，将这个数据包发送到 目标服务器(192.168.56.53)的8080端口。并且连接 辅助服务器（192.168.56.12），请求上面的intercept 返回响应数据包给它
     [root@nfs-client2 sbin]#  route  add -net 192.168.50.0 netmask 255.255.255.0 gw 192.168.56.12
     #<= 设置所有到192.168.50.x的响应请求 通过路由 转发到辅助服务器

（3）辅助服务器（assistant server 192.168.56.12）

    [root@linux-node2 sbin]#  intercept -i eth0 -F 'tcp and src port 8080' -d
	#<=intercept 监听本地eth0的8080端口。来捕获tcp应用程序的数据包