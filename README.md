# nginx_reading_notes
Reading notes of Understanding nginx


chapter1.3.4 Linux内核参数的优化 
--------------------------------
>一、修改内核参数的方法：
>>(1)通过/et/sysctl.conf文件；
>>>修改/et/sysctl.conf对应参数；sysctl -p。
-p[FILE], --load[=FILE] Load  in  sysctl  settings  from  the file specified or /etc/sysctl.conf if none given. 

>>(2)直接通过proc文件系统修改。

>二、NGINX优化涉及到的常用内核参数：
>>(1)fs.file-max
>>>最大并发数

>>Q:为什么最大fd数和并发相关呢？

>>>每一个来自client 的ingress connection都要占用一个fd,如下图所示：
![image](https://raw.githubusercontent.com/dahaiyu/nginx_reading_notes/master/img_folder/chapter1/lsof_fd.png) 
每一个ingress connection并不会新占用一个本机端口号,如下图所示：
![image](https://github.com/dahaiyu/nginx_reading_notes/blob/master/img_folder/chapter1/netstat_1.png?raw=true) 
即socket accept一个新链接时，Local Address不需要占用一个新端口号，只需要占用一个fd。

>>Q:Server端所能接受的ingress connections是否受限于端口号(65535)？
  
>>（2）net.ipv4.tcp_tw_recycle
  
>>>Setting tcp_tw_recycle to 1 makes a Linux host drop TIME_WAIT connections much faster.  Instead of a predefined 2*MSL period 
    of 60s, the host will use a timeout based on RTT estimate.  For LANs, it is usually several milliseconds. 
    
>>（3）net.ipv4.tcp_tw_reuse
  
>>>Setting tcp_tw_reuse to 1 will make a host reuse the same connection quickly for outgoing connections. 
    
>>（4）net.ipv4.tcp_fin_timeout - INTEGER
  
>>>@Time to hold socket in state FIN-WAIT-2, if it was closed by our side. Peer can be broken and never close its side,
    or even died unexpectedly. Default value is 60sec.Usual value used in 2.2 was 180 seconds, you may restore
    it, but remember that if your machine is even underloaded WEB server,you risk to overflow memory with kilotons of dead sockets,
    FIN-WAIT-2 sockets are less dangerous than FIN-WAIT-1,because they eat maximum 1.5K of memory, but they tend to live longer.	
    
>>@The length  of  time  in  seconds  it  takes to receive a final FIN before the socket is  always  closed.  
    This  is  strictly  a violation  of  the  TCP specification, but required to prevent denial-of-service attacks.
  
>>Q:从上段文字可知，当client主动关闭，若迟迟没有收到server端的FIN, 则client 的传输控制块status什么时候会变为CLOSED?

>>Q:参考下图，回答问题：TCP 双方同时关闭时status transition?什么时候进行CLOSING状态？什么时候双方同时进入TIME_WAIT？
![image](https://github.com/dahaiyu/nginx_reading_notes/blob/master/img_folder/chapter1/tcp_status.png?raw=true)

>>（5）et.ipv4.tcp_max_tw_buckets
>>>/proc/sys/net/ipv4/tcp_max_tw_buckets: Maximal number of timewait sockets held by the system simultaneously. If this number is exceeded time-wait socket is immediately destroyed and a warning is printed. This limit exists only to prevent simple DoS attacks, you must not lower the limit artificially, but rather increase it (probably, after increasing installed memory), if network conditions require more than the default value.

>>(6)常用配置选项:daemon、master_process on|off、error_log path、debug_connect IP|CIDR、worker_rlimit_*(core\nofile\sigpending)、
env VAR|VAR=VALUE、work_processces、work_connections、log_file、accept_mutex、accept_mutex_delay、multi_accept、use、
listen address:port [default_server(higher priority than server_name)|backlog=num|recvbuf=size|sndbuf=size|deferred|bind|ssl]、
server_name(可以匹配多个时的匹配顺序)、
