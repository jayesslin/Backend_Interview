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