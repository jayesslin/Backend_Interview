linux 常用操作指令

1. 如何看当前linux系统有几颗物理CPU和没颗CPU的核数

		cat /proc/cpuinfo|grep -c 'physical id' --> 4
		cat /proc/cpuinfo|grep -c 'processor' --> 4

2. 查看系统负载有两个常用的命令，是哪两个？这三个数值表示什么含义呢？

		[root@centos6 ~ 10:56 #37]# w
		10:57:38 up 14 min,  1 user,  load average: 0.00, 0.00, 0.00
		USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
		root     pts/0    192.168.147.1    18:44    0.00s  0.10s  0.00s w
		[root@centos6 ~ 10:57 #38]# uptime
		10:57:47 up 14 min,  1 user,  load average: 0.00, 0.00, 0.00

	其中load average即系统负载，三个数值分别表示一分钟、五分钟、十五分钟内系统的平均负载，即平均任务数。
	

3. linux系统里，您知道buffer和cache如何区分吗？

	buffer和cache都是内存中的一块区域，当CPU需要写数据到磁盘时，由于磁盘速度比较慢，所以CPU先把数据存进buffer，然后CPU去执行其他任务，buffer中的数据会定期写入磁盘；当CPU需要从磁盘读入数据时，由于磁盘速度比较慢，可以把即将用到的数据提前存入cache，CPU直接从Cache中拿数据要快的多。
	
4. 使用top查看系统资源占用情况，以及其参数
	
	1. VIRT虚拟内存用量
	2. RES物理内存用量
	3. SHR共享内存用量
	4. %MEM内存用量

5. 使用ps查看系统进程
	
	ps -aux 或者ps -elf
	
	获得pid 可 kill
	
6. ps 查看系统进程时，有一列为STAT， 当前进程的stat为Ss表示的含义，Z表示的含义
	
	S表示正在休眠；s表示主进程；Z表示僵尸进程。
	
7. 查看系统开启的端口

	netstat -lnp
	
	netstat -an 查看网络连接的情况
	
8. 修改ip配置文件并重启网卡

	使用vi或者vim编辑器编辑网卡配置文件*/etc/sysconfig/network-scripts/ifcft-eth0*（如果是eth1文件名为ifcft-eth1）
	
	修改网卡后，可以使用命令重启网卡：

		ifdown eth0
		ifup eth0
		
	也可以重启网络服务：

		service network restart
		
9. Linux中最多可以有多少个进程？
	
		ulimit -u 
		
		linyandeMacBook-Pro:~ linyan$ ulimit -u
		709

	这属于软限制，是可以改变的。也就是说在我的机器上最多可以有4096个进程，但是我可以通过改变这个参数的值来修改对于进程数量的软限制，比如说用下面的命令将软限制改到5120。
		
		 ulimit -u 5120
		
10. 一个进程中最多可以有多少个线程？

	在上一篇文章Linux中线程占用内存中，我们知道了创建一个线程会占用多少内存，这取决于分配给线程的调用栈大小，可以用ulimit -s命令来查看大小（一般常见的有10M或者是8M）。我们还知道，一个进程的虚拟内存是4G，在Linux32位平台下，内核分走了1G，留给用户用的只有3G，于是我们可以想到，创建一个线程占有了10M内存，总共有3G内存可以使用。于是可想而知，最多可以创建差不多300个左右的线程。

	因此，进程最多可以创建的线程数是根据分配给调用栈的大小，以及操作系统（32位和64位不同）共同决定的。

11. 提高linux效率的命令，常见vim操作

12. linux写时复制


		