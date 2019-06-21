# 系统扩展的方式 

1. scaling up  增强机器 
2. scaling out 向外扩展，增加 机器

## 集群： 组合多个服务器（计算机） 合成的单个系统

Lvs 做请求流量调度，  接收vip（虚拟ip，统一对外ip）， 转发到各个服务器（server IP）

## Linux cluster 类型

1. 负载均衡集群 LB ： Load balance
2. 高可用集群 High availibility  高可用 SPOF （single point OF Failure）
	+ MTBF : Mean Time Between Failure 平均无故障时间
	+ MTTR : Mean Time to Restoration 平均恢复前时间（故障时间）
	
	> A = MTBF / （MTBF + MTTR） (0,1)之间， 数值越高，可用性越高 （无故障时间占的比例）
3.	高性能集群 HPC ： high performance computing 高性能 （比如计算天气预报）


## 分布式储存：  fastdfs


# LB cluster 的实现：
* 硬件
	1. F5 Big——IP
	2. citrix netscaler
	3. A10 A10
* 软件
	1. lvs
	2. 	ngnix： 支持7层调度
	3. 	haproxy : 支持7层调度
	4. 	ats ： apache traffic server， yahoo 捐助 
## 基于工作的协议层次划分
### 传输层（通用） 目标端口转换： DPRORT， （
##### DNAT协议？DNAT 目标地址转换，用户的请求发网同一公网地址，只能转换到内网的一个服务器上）
	- lvs
	- niginx ： stream
	- haproxy：mode tcp
### 应用层（专用） ： 针对特定协议，自定义的请求模式



# session 会话保持。 （session 一致性） （负载均衡）
- session sticky ： 同一用户调度固定服务器
		根据IP地址，hash，	（可能因为某些公司 统一客户端Ip（NAT），造成单一服务器负载过大）
- session replication： 每台服务器都备份全部session  内网贷款负载
- session server： 专门的session 服务器
		Memcached， Redis

# 高可用集群实现方案
*  keepalived：地址漂移 （vrrp协议？）


# LVS : 集成在linux内核 ， 
* VS  virtual server 负责调度
* RS real server 负责提供服务
* L4 四层路由交换机
	
## 工作原理： VS 根据请求报文的目标IP和目标协议以及端口将其调度转发至某RS， 根据调度算法来挑选RS

- VS ： 调度器， 负载均衡
- Real server（lvs）， upstream Server(nginx
- backend server(haproxy)

### 访问流程： CIP（客户端ip） 《==》 VIP （vs外网的IP） === DIP （VS内网的IP） 《==》 RIP（Real Server IP）

# LVS集群的类型
1. lvs ： ipvsadm/ipvs
	- ipvsadm ： 用户空间的命令行工具， 管理集群服务以及realserver
	- ipvs ： 工作于内核空间netfilter的input **钩子上** 的框架
2. lvs 集群类型（模式）
	- lvs-NAT ： 

		> 修改请求报文的目标ip ，多目标ip的DNAT　
		- nat的工作原理，（lvs来回压力大） 支持端口映射， RIP可以不在一个ip网段（用路由或者交换机就可以），
		- ![](http://ww4.sinaimg.cn/large/006tNc79ly1g471ngzhizj316a0u041a.jpg) 
		- NAT 模式
		- ![](http://ww4.sinaimg.cn/large/006tNc79ly1g471usjwdsj314q0obe4d.jpg)

		
	- lvs-DR 

		> 相比于nat模式，减少了lvs服务器负载），请求报文都经过lvs，响应报文不需要， 可以支持更多的real server

		- 直接路由，LVS默认模式，通过为请求报文重新封装一个MAC首部进行转发，源MAC是DIP所在的接口的MAC， 目标MAC是某挑选出的RS的RIP所在的接口的MAC地址；源IP/PORT，以及目标IP/PORt均保持不变（及不支持端口映射）。
		- 图解， LVS 的 VIP 需要配置 RS 群的MAC地址， 通过Director（交换机）广播找。并且确保前端路由器绑定VIP和DIRECTOR的MAC地址
		- 在RS上修改内核参数以限制arp通告以及应答级别（arp_ignore, arp_announce (不搭理，不发布自己的ip地址）） 
		- 或者用Arptables工具.....
		-  ![](http://ww3.sinaimg.cn/large/006tNc79ly1g472le0oxbj31a00u0e83.jpg)

		
	- lvs-tun（隧道模式）
	
		>  跨异地， 跨机房， 不同网段
		- 修改IP报文头部（lvs 转发给rip，rip存有vip）
		- 回应直接返回给cip
		- 不支持端口映射
		- 需要特殊配制， IP报文
		- ![](http://ww2.sinaimg.cn/large/006tNc79ly1g47eo9y8gbj318o0u0b2a.jpg)
		
	- lvs-fullnat
		
		> 通过同时修改请求报文的源IP地址和目标IP地址进行转发
		- CIP ---> DIP
		- VIP ---> RIP
		- 请求响应都经过Director
		- 支持端口映射
		- kernel默认不支持

	- 总结对比
	
		|   | VS/NAT | VS/TUN | VS/DR | 
		| :----: |:----:| :----: |:----: |
		| Server  | any | Tunneling |Non-arp Device |
		|Server network | private | LAN/WAN |LAN |
		|Server number | low(10~20) | high(100) |high(100) |
		|Server gateway | load balance | own router|own router |


# 调度算法

## 静态调度
 
> 不考虑后端服务器状态
	
1. RoundRobin 轮询
	
2. WeightRoundRobin 加权轮询
	
3. Source Hashing , 实现session sticky，源IP地址hash， 实现了会话绑定
	
4. Destination Hashing 目标地址哈希， 将发往同一个目标地址的请求始终转发至第一次挑中的RS，典型使用场景是正向代理缓存场景中的负载均衡。
	
	>	如： 宽带运营商
		
		
## 动态调度
> 主要根据每RS当前的负载状态及调度算法进行调度
> 
> Overhead = value 较小的 RS 将被调度、

1. 最小连接 least connections 

	适用于长链接应用（缺点： 没有考虑权重）
	overhead值越小 ， 优先调度
	>Overhead = activeconns（活动连接） * 256 + inactiveconns（不活动链接，光tcp）
	> 
	> 一个活动链接 = 256 个 不活动链接
	
2. 权重最小链接 Weight least connections
	
	**默认调度方法**
	>Overhead = （activeconns（活动连接） * 256 + inactiveconns（不活动链接，光tcp））/ **weight**

3. SED：Shortest Expection Delay，初始链接高权重优先
	
	>Overhead = (activeconns +1 ) * 256 / **weight**
	
4. NQ: Never Queue 第一轮均匀分配， 后续SED

5. LBLC： Locality-Based LC, 动态的DH算法

	>使用场景： 根据负载均衡状态实现正向代理
	
6. LBLCR： LBLC with Replication ， 带复制功能的LBLC

	>解决LBLC负载不均衡问题， 从负载重的复制到负载轻的RS

	*注：5，6 宽带运营商常用*








