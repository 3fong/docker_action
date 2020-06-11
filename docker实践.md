### docker实践 ###


### 问题点 ###


依赖服务是否要镜像化 考虑部署运维成本 
依赖服务未镜像化 与容器服务通讯问题
文件持久化 minio fastdfs(文件归属控制,权限控制的粒度更细)

时区问题 中文字体问题



3台机器,网络互通 资源配额

架构方案:
单机容器,架构图:
入口层:通一流量控制
服务发现层:consul containerpilot管理容器生命周期,辅助consul进行服务管理
集群层:集成宿主机 docker swarm flannl进行网络打通,便于集群部署
容器层:docker宿主机,部署容器
基础服务层: 数据库 中间件
域名解析服务: Triton Container Name Service(选用)

containerpilot:容器间通讯问题(不支持跨语言流量调度处理);
服务联调问题


容器调度用docker compose

需要增加镜像仓库,文件服务器,服务监控,通一的日志服务(日志监控)

集群版本的化使用docker swarm


#### 服务架构 ####
分层:
	体系架构分层 数据状态 持久化数据(文件) 数据库 缓存
	软件业务分层 mvc

入口层    
	统一入口 openresty 流量管理 调度   

业务层    
	容器化日志监控 整个业务监控 日志丢失问题
	容器运行多少应用 模型拆不开(本地环境的模型 单机环境共生模型) 需要单体方式部署 phthoy容器 多组件依赖 最好在一起放   

存储层    
	文件持久化 数据存储 
	怎么访问 怎么处理 地址管理(部署ip端口)  

#### 服务设计 ####
- ci,cd   

开发自测通过,自动部署到测试环境,验证通过后自动部署到生产环境   
单元,集成,端对端测试 jmock   
jenkins pipline
drone.io
heroku https://devcenter.heroku.com/articles/procfile

version reback???

- 镜像仓库

用于镜像存储和访问.功能: 权限控制 镜像同步

技术选型:
docker hub:共有仓库 镜像构建不涉及业务代码 等镜像下载后导入业务代码 实现镜像和业务代码分离
缺点: 快速切换需要考虑环境,业务代码版本管理整合问题 
docker registry
harbor 
蜻蜓 

- 资源调度

用于镜像分发.功能:集群对接,集群权限管理,成本核算,环境初始化等
宿主机环境初始化:
	vagrant:初始化虚拟机
	chef,puppet,ansible,salt初始化物理机.
docker machine


- 容器调度

用于判断哪些宿主机创建容器.功能:主机过滤 调度策略 
fleet: 调度器和集群管理工具
marathon:
swarm
mesos:宿主机抽象服务,用于为调度器联合宿主机资源
kubernetes:容器组管理工具,可以通过标签,分组和配置通信子网
compose:通过定义配置文件来提供容器组管理功能.通过docker的link分析容器依赖关系.


- 容器编排

容器创建后对外提供服务方式
docker compose管理 建议有镜像私服   

服务发现
etcd:服务发现/全局分布式键值对存储
consul
zookeeper
crypt:加密的etcd条目的项目
confd:观测键值对存储变更和新值的触发器重新配置服务

- 容器网络
	
本地网络:容器间,容器与宿主间通讯,通过虚拟接口配置,子网,iptables,nat管理实现
	为容器间通讯提供了两种方案:
	公开容器端口.
	links

覆盖网络:构造跨宿主机的通一网络
外部网络工具:
	flannel:覆盖网络提供给每个宿主机一个独立子网
	weave:覆盖网络描述一个网络上的所有容器
	pipework:一个高级网络工具,它用于任意高级网络配置
	tinc:轻量vpn软件.采用隧道和加密实现.

#### 容量规划 ####
监控当前服务 资源占用情况
对比docker服务

- 容量评估   

a. 选择合适的压测指标 

    	系统类指标: cpu使用率,内存占用量,磁盘I/O使用率,网卡带宽
    	服务类指标:接口响应平均耗时,P999耗时,错误率
b. 获取单机最大容量(线上实施)
 
    	集群最大容量:单机最大容量*集群数量
    	单机压测:日志回放;TCP-Copy;两种方式模拟请求(不能进行编辑操作,有缺口)
    	集群压测:摘取节点,增加单机访问量(更准确)
    	QPS+区间加权.增加结果的准确性
c. 实时获取集群的运行负荷

    	负荷是判断扩容的基础.
    	统计单机的不同耗时区间内请求数.然后汇总,进行区间加权,算出集群负荷

- 调度决测

    通过水位线进行调度,分为安全线,致命线   
    1. 扩容 按数量,按比例
    2. 缩容 还要注意缩容引起的网络抖动可能会触发扩容.实际扩容可以采用多点采集,过半数才进行扩容的方式

#### 节点检测 ####

uptime robot:检测健康检查和断点
new relic:服务器和应用程序监视服务.主要是服务器监视
google analytics:网站分析跟踪工具
statsd: 通过udp,tcp监听各种统计信息,包括计数器,定时器等;
VictoriaMetrics: prometheus 集群的监控以及报警方案

#### 调试技术 ####
strace gbd /proc
nsenter
nsinit

#### 成本 ####

硬盘 内存 cpu 网络带宽(内外网) 
第三方依赖 数据库 中间件 第三方api 额外服务(指纹识别问题) 连接状态 主动连接被动连接(容器开通随机端口ftp)



#### 价值 ####
建立资源池
资源利用率提高
快速部署构建


#### 配置规划 ####
容器分区、系统内核、Docker版本、Docker Daemon访问控制及日志审计


容器分区
#### 日志收集 ####

logstash日志聚合工具

logstash server: clay/journalist-public
logstash-forwarder server: clay/scribe-public
日志索引


#### tag ####
 version information, intended destination (prod or test, for instance), stability, or other information 


One case where it is appropriate to use bind mounts is during development, when you may want to mount your source directory or a binary you just built into your container. For production, use a volume instead, mounting it into the same location as you mounted a bind mount during development.



#### docker swarm ####

用于创建docker主机集群的工具.



#### clay构建流程 ####

代码合并到master分支
用git tag打上发布标签
构建docker镜像
用docker tag打上发布标签
将docker镜像推送到registry
更新集群容器版本
确认备用容器正在运行
升级主容器
等待新的主容器上线
升级备用容器
更新生成集群容器版本

问题:为什么不直接启动新的主容器,然后干掉就主容器?端口占用问题?


#### 架构方案二 ####
mesos + marathon + docker
mesos: 提供伸缩性服务
marathon: 集群初始化和控制
docker:


#### 入口层 ####

前后端分离,需要再进行聚合:

golang,nodejs:静态资源嵌入代码里
java: web jar模式:版本化;动态处理;

golang:可执行文件;虚拟文件系统;

版本化的对应关系问题: 前后端分离后,版本的匹配问题.
1 团队协作
2 进行应用适配
3 前端在后端的项目里,聚合运行.


openresty:解决统一域名,区分直销分销同时访问

web: api调用和静态资源
cookie存储状态,适配后续请求.
考虑点:
	第一次登录
	后续登录


单点登录:

	支持标准:哪类标准,openid,oid,oauth2;
	登录登出;
	会话管理;
	用户信息获取,不同库源;

连库,数据库模型;
openresty只是配置登录页面跳转;



github flow




#### 压测: ####

核心点:出问题,不能看指标,指标是结果;要找原因.
代码,调用链很复杂.工具只是辅助,要找到实际问题是什么.
debug包 获取详细日志 


perf:库级别调用链.应用层面还需要专门的工具进行分析.


压测:

逐渐加压,看看能到多少访问
并发数,总访问量,keepalive
获取瓶颈点,热点代码,便于分析和调整
官方工具,debug包,通过火焰图分析具体请求
wrk
如果对于性能有要求,可以将tcp改成unit套接字,提高2,3倍性能

热点代码分析:

1 具体问题分析,如果是后端服务问题,就做lb,进行分流处理




#### 业余扩展 ####

Java配置热更新














