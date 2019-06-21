#Redis高可用以及分布式锁
**redis 是 单进程单线程模型 ，保证了原子性**


*锁*

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