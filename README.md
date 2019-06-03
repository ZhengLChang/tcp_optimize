优化/etc/sysctl.conf

#net.ipv4.tcp_mem  单位为页 根据实际修改,下面为1G内存的机器

net.ipv4.tcp_mem = 9162	22170	128000

#这个参数表示进程可以同时打开的最大句柄数，这个参数直接限制最大并发连接数。

fs.file-max = 999999

#这个参数表示当服务器主动关闭连接时，socket保持在FIN-WAIT-2状态的最大时间

net.ipv4.tcp_fin_timeout = 30

#这个参数设置为1,表示允许将TIME-WAIT状态的socket重新用于新的TCP链接。这个对服务器来说很有意义，因为服务器上总会有大量TIME-WAIT状态的连接

net.ipv4.tcp_tw_reuse = 1

使/etc/sysctl.conf修改立刻生效命令：/sbin/sysctl -p

/etc/security/limits.conf

*	soft nofile 65536

*	hard nofile 65536


理论基础：

	每一个TCP连接都会有对应的socket封装，而每个socket都要占用一个fd，现在的业务系统大都采用epoll的网络I/O模型，他可以高效的处理大批量socket句柄，而这个socket句柄的对应的TCP读写缓存再加上一个TCP控制块就是单个TCP连接所消耗的内存，当然这个读写缓存的大小是根据系统的需要动态变化的，和TCP的滑动窗口大小成正相关。

	对于tcp能够使用多少缓存，linux是会有全局控制的,系统内存(4G)

	TCP能够使用的内存：这三个值就是TCP使用内存的大小，单位是页，每个页是4K的大小，如下(这个可以在sysctl.conf中net.ipv4.tcp_mem中进行调整)：

	cat /proc/sys/net/ipv4/tcp_mem 

	187506	250009	375012

	这三个值分别代表

	Low：187506   (187506*4/1024/1024大概0.71G)

	Pressure：250009 (250009*4/1024/1024大概0.95G)

	High：375012   (375012*4/1024/1024大概1.43)

	这个也是系统装后的默认取值，也就是说最大有1.43个g（35%的内存）可以用作TCP连接，这三个量也同时代表了三个阀值，TCP的使用小于第二个值时kernel不会有任何提示操作，当大于第二个值时进入压力模式，当高于第三个值时将不接受新的TCP连接，同时会报出“Out  of  socket memory”或者“TCP:too many of orphaned sockets”。该系统显然有修改空间


	TCP读缓存大小，单位是字节：第一个是最小值4K，第二个是默认值85K，第三个是最大值6M，如下(这个可以在sysctl.conf中net.ipv4.tcp_rmem中进行调整)：

	cat /proc/sys/net/ipv4/tcp_rmem 

	4096	87380	6291456


	TCP写缓存大小，单位是字节：第一个是最小值4K，第二个是默认值16K，第三个是最大值4M，如下(这个可以在sysctl.conf中net.ipv4.tcp_wmem中进行调整)：

	cat /proc/sys/net/ipv4/tcp_wmem 

	4096	16384	4194304

