#Redis高可用以及分布式锁
**redis 是 单进程单线程模型 ，保证了原子性**
Redis的高并发和快速原因

1. redis是基于内存的，内存的读写速度非常快；

2. redis是单线程的，省去了很多上下文切换线程的时间；

3. redis使用多路复用技术，可以处理并发的连接。非阻塞IO 内部实现采用epoll，采用了epoll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，绝不在io上浪费一点时间。

IO多路复用技术

- redis 采用网络IO多路复用技术来保证在多连接的时候， 系统的高吞吐量。

- 多路-指的是多个socket连接，复用-指的是复用一个线程。多路复用主要有三种技术：select，poll，epoll。epoll是最新的也是目前最好的多路复用技术。

- 这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络IO的时间消耗），且Redis在内存中操作数据的速度非常快（内存内的操作不会成为这里的性能瓶颈），主要以上两点造就了Redis具有很高的吞吐量。

# redis 数据结构

- redis 数据结构使用场景



1. String——字符串
	
2. Hash——字典
	
3.  List——列表

4.  Set——集合

5. Sorted Set——有序集合

下面我们就来简单说明一下它们各自的使用场景：

## String——字符串

String 数据结构是简单的 key-value 类型，value 不仅可以是 String，也可以是数字（当数字类型用 Long 可以表示的时候encoding 就是整型，其他都存储在 sdshdr 当做字符串）。使用 Strings 类型，可以完全实现目前 Memcached 的功能，并且效率更高。还可以享受 Redis 的定时持久化（可以选择 RDB 模式或者 AOF 模式），操作日志及 Replication 等功能。除了提供与 Memcached 一样的 get、set、incr、decr 等操作外，Redis 还提供了下面一些操作：
	

	1.LEN niushuai：O(1)获取字符串长度
	2.APPEND niushuai redis：往字符串 append 内容，而且采用智能分配内存（每次2倍）
	3.设置和获取字符串的某一段内容
	4.设置及获取字符串的某一位（bit）
	5.批量设置一系列字符串的内容
	6.原子计数器
	7.GETSET 命令的妙用，请于清空旧值的同时设置一个新值，配合原子计数器使用

## Hash——字典
在 Memcached 中，我们经常将一些结构化的信息打包成 hashmap，在客户端序列化后存储为一个字符串的值（一般是 JSON 格式），比如用户的昵称、年龄、性别、积分等。这时候在需要修改其中某一项时，通常需要将字符串（JSON）取出来，然后进行反序列化，修改某一项的值，再序列化成字符串（JSON）存储回去。简单修改一个属性就干这么多事情，消耗必定是很大的，也不适用于一些可能并发操作的场合（比如两个并发的操作都需要修改积分）。而 Redis 的 Hash 结构可以使你像在数据库中 Update 一个属性一样只修改某一项属性值。

> 存储、读取、修改用户属性

## List——列表
List 说白了就是链表（redis 使用双端链表实现的 List），相信学过数据结构知识的人都应该能理解其结构。使用 List 结构，我们可以轻松地实现最新消息排行等功能（比如新浪微博的 TimeLine ）。List 的另一个应用就是消息队列，可以利用 List 的 *PUSH 操作，将任务存在 List 中，然后工作线程再用 POP 操作将任务取出进行执行。Redis 还提供了操作 List 中某一段元素的 API，你可以直接查询，删除 List 中某一段的元素

> 1.微博 TimeLine
>2.消息队列

## Set——集合
Set 就是一个集合，集合的概念就是一堆不重复值的组合。利用 Redis 提供的 Set 数据结构，可以存储一些集合性的数据。比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。因为 Redis 非常人性化的为集合提供了求交集、并集、差集等操作，那么就可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

> 1.共同好友、二度好友
> 2.利用唯一性，可以统计访问网站的所有独立 IP
> 3.好友推荐的时候，根据 tag 求交集，大于某个 threshold 就可以推荐

## Sorted Set——有序集合

和Sets相比，Sorted Sets是将 Set 中的元素增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，比如一个存储全班同学成绩的 Sorted Sets，其集合 value 可以是同学的学号，而 score 就可以是其考试得分，这样在数据插入集合的时候，就已经进行了天然的排序。另外还可以用 Sorted Sets 来做带权重的队列，比如普通消息的 score 为1，重要消息的 score 为2，然后工作线程可以选择按 score 的倒序来获取工作任务。让重要的任务优先执行。

> 1.带有权重的元素，比如一个游戏的用户得分排行榜
> 2.比较复杂的数据结构，一般用到的场景不算太多

### 为什么zset（redis）里用跳表不用红黑树

简单来说： 

1. skiplist的复杂度和红黑树一样，而且实现起来更简单。

2. 在并发环境下skiplist有另外一个优势，红黑树在插入和删除的时候可能需要做一些rebalance的操作，这样的操作可能会涉及到整个树的其他部分，而skiplist的操作显然更加局部性一些，锁需要盯住的节点更少，因此在这样的情况下性能好一些。


# redis持久化策略之一

RDB方式 

RDB方式的持久化是通过快照（snapshotting）完成的，当符合一定条件时Redis会自动将内存中的所有数据进行快照并存储在硬盘上。进行快照的条件可以由用户在配置文件中自定义，由两个参数构成：时间和改动的键的个数。当在指定的时间内被更改的键的个数大于指定的数值时就会进行快照。RDB是Redis默认采用的持久化方式，在配置文件中已经预置了3个条件：
 
save 900 1 

save 300 10 

save 60 10000

save参数指定了快照条件，可以存在多个条件，条件之间是“或”的关系。如上所说，save900 1的意思是在15分钟（900秒钟）内有至少一个键被更改则进行快照。如果想要禁用自动快照，只需要将所有的save参数删除即可。 

Redis默认会将快照文件存储在当前目录的dump.rdb文件中，可以通过配置dir和dbfilename两个参数分别指定快照文件的存储路径和文件名。 

理清Redis实现快照的过程对我们了解快照文件的特性有很大的帮助。快照的过程如下。
 
（1）Redis使用fork函数复制一份当前进程（父进程）的副本（子进程）； 

（2）父进程继续接收并处理客户端发来的命令，而子进程开始将内存中的数据写入硬盘中 

的临时文件； 

（3）当子进程写入完所有数据后会用该临时文件替换旧的RDB文件，至此一次快照操作完成。 

在执行fork的时候操作系统（类Unix操作系统）会使用写时复制（copy-on-write）策略，即fork函数发生的一刻父子进程共享同一内存数据，当父进程要更改其中某片数据时（如执行一个写命令），操作系统会将该片数据复制一份以保证子进程的数据不受影响，所以新的RDB文件存储的是执行fork一刻的内存数据。 

通过上述过程可以发现Redis在进行快照的过程中不会修改RDB文件，只有快照结束后才会将旧的文件替换成新的，也就是说任何时候RDB文件都是完整的。这使得我们可以通过定时备份RDB文件来实现Redis数据库备份。RDB文件是经过压缩（可以配置rdbcompression参数以禁用压缩节省CPU占用）的二进制格式，所以占用的空间会小于内存中的数据大小，更加利于传输。 

除了自动快照，还可以手动发送SAVE或BGSAVE命令让Redis执行快照，两个命令的区别在于，前者是由主进程进行快照操作，会阻塞住其他请求，后者会通过fork子进程进行快照操作。 

Redis启动后会读取RDB快照文件，将数据从硬盘载入到内存。根据数据量大小与结构和服务器性能不同，这个时间也不同。通常将一个记录一千万个字符串类型键、大小为1GB的快照文件载入到内存中需要花费20～30秒钟。 

通过RDB方式实现持久化，一旦Redis异常退出，就会丢失最后一次快照以后更改的所有数据。这就需要开发者根据具体的应用场合，通过组合设置自动快照条件的方式来将可能发生的数据损失控制在能够接受的范围。如果数据很重要以至于无法承受任何损失，则可以考虑使用AOF方式进行持久化。


# *锁*

- 分布式锁
	- 为什么需要锁
		1. 多任务环境，多线程
		2. 任务都需要对同一共享资源进行写操作
		3. 对资源访问都是互斥的
		
		> 1⃣️： 竞争🔒  ，2⃣️：占有🔒 ，3⃣️： 任务阻塞，4⃣️：释放🔒
		
	---
	
	- 分布式锁
	
		1. 缓存有效期：当key过期时，他会被删除 （“PX”命令）
		**有效期设置，两倍正常业务时间**
		2. SETNX命令： SETNX key value， 将key的值设为value，当且仅当key不存在。如果key存在，不做操作。「set if not exists」（“NX”命令）
		3. Lua脚本： 支持redis操作序列的原子性


		- 加锁
			> 通过setNX向特定的key写入一个随即值，并同时设置失效时间，写值成功既加锁成功
			
			- 注意： 
			 	1. 必须给锁设置一个失效时间 ==〉避免死锁
			 	2. 加锁时，每个节点产生一个随机字符串 ==〉避免锁误删(时间失效后误删别的进程锁)
			 	3. 写入随即值与设置失效时间必须同时 ==〉保证加锁是原子性
			 	
					```SET key value NX PX 30000```
			
		- 解锁
			> 匹配随机值，删除redis上的特点key数据，**要保证获取数据/判断一致以及删除数据操作是原子的**
			
			执行的lua脚本如下：
			
				if redis.call("get",KEYS[1]) == ARGV[1] then
					return redis.call("del",KEYS[1])
				else
					return 0
				end		
	- 代码实例
	> java中redislock 继承Lock
		
	- 非阻塞式加锁，使用setNX命令返回ok的枷锁成功，并产生随机值
		
		
				private static final String KEY = "KEY";
				//通过theadlocal传值 
				private ThreadLocal<String> local =new ThreadLocal<String>();
				
			
		- 加锁
				
					public boolean tryLock(){
					    	//产生随机值
					    	String uuid = UUID.randomUUID().toString();
					    	//获取redis连接 ,**只有原始链接才能使用SETNX命令**
					    	Jedis jedis = (jedis) factory.getConnection().getNativeConnection();
					    	//使用setNX命令请求写值，设置失效时间
					    	String ret = jedis.set(KEY,uuid,"NX","PX",1000);
					    	if (ret.equals("OK")){
					    		//同时通过threadlocal设置线程变量
					    		local.set(uuid);
					    		return true;
					    	}
					    	else{
					    		return false;
					    	}
					 }
					 
					 
		- 解锁（读取lua脚本）
			
					public  boolean unlock(){
						//读取lua脚本
						String script = Fileutils.readFileByLines("文件路径，类似上面的.lua脚本");
						//获取redis连接 
				    		Jedis jedis = (jedis) factory.getConnection().getNativeConnection();
					    	//通过原始链接链接redis执行lua脚本
					    	//threadlocal 取得uuid的字符串值
					    	jedis.eval(script, Arrays.asList(KEY),Arrays.asList(local.get()));
					}
	
	




*集群*

redis 并发量 10w左右

- Redis 集群方案比较
	
	- 哨兵模式

	![](http://ww1.sinaimg.cn/large/006tNc79ly1g49e6k7wkaj30ga0hc0w1.jpg)
	 >  借助哨兵sentinel（也可以多个哨兵，保证多可用）工具来监控master节点的状态， 如果master节点异常，则会从slave中切换一台作为master。
	 > 主从切换会有 <span style= “color:red">瞬断</span> 的问题。
	 
	 哨兵缺点和单点并发相同， 并发能力不行，对外的写只有一台master
	 
	-------
	
	- 高可用集群模式
	
	![](http://ww4.sinaimg.cn/large/006tNc79ly1g49ere2rgmj31570u0dio.jpg)
	
	> jedisCLuster 具有负载均衡的作用
	
	高可用集群具有复制，高可用和分片特性。可水平扩展，线性扩展到1000节点（理论上1000*1万并发）