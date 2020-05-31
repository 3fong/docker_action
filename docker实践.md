### docker实践 ###

#### 配置规划 ####
容器分区、系统内核、Docker版本、Docker Daemon访问控制及日志审计


容器分区
日志收集

#### tag ####
 version information, intended destination (prod or test, for instance), stability, or other information 


One case where it is appropriate to use bind mounts is during development, when you may want to mount your source directory or a binary you just built into your container. For production, use a volume instead, mounting it into the same location as you mounted a bind mount during development.


3台机器,网络互通 资源配额



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
docker machine


- 容器调度

用于判断哪些宿主机创建容器.功能:主机过滤 调度策略 
fleet: 调度器和集群管理工具
marathon:
swarm
mesos:宿主机抽象服务,用于为调度器联合宿主机资源
kubernetes:容器组管理工具,具有出色的调度能力
compose:容器组编排工具


- 容器编排

容器创建后对外提供服务方式
docker compose管理 建议有镜像私服   
服务发现
etcd:服务发现/全局分布式键值对存储
concul
zookeeper
crypt:加密的etcd条目的项目
confd:观测键值对存储变更和新值的触发器重新配置服务

- 容器网络
	容器间,容器与宿主间通讯
docker本地网络为容器间通讯提供了两种方案:
	公开容器端口.
	links

外部网络工具:
	flannel:覆盖网络提供给每个宿主机一个独立子网
	weave:覆盖网络描述一个网络上的所有容器
	pipework:一个高级网络工具,它用于任意高级网络配置

- 容量规划

容量评估
1. 选择合适的压测指标 
	系统类指标: cpu使用率,内存占用量,磁盘I/O使用率,网卡带宽
	服务类指标:接口响应平均耗时,P999耗时,错误率
2. 获取单机最大容量(线上实施)
	集群最大容量:单机最大容量*集群数量
	单机压测:日志回放;TCP-Copy;两种方式模拟请求(不能进行编辑操作,有缺口)
	集群压测:摘取节点,增加单机访问量(更准确)
	QPS+区间加权.增加结果的准确性
3. 实时获取集群的运行负荷
	负荷是判断扩容的基础.
	统计单机的不同耗时区间内请求数.然后汇总,进行区间加权,算出集群负荷

调度决测
	通过水位线进行调度,分为安全线,致命线
1. 扩容 按数量,按比例
2. 缩容 还要注意缩容引起的网络抖动可能会触发扩容.实际扩容可以采用多点采集,过半数才进行扩容的方式



### 问题点 ###


依赖服务是否要镜像化 考虑部署运维成本 
依赖服务未镜像化 与容器服务通讯问题
文件持久化 minio fastdfs(文件归属控制,权限控制的粒度更细)

时区问题 中文字体问题




#### 成本 ####

硬盘 内存 cpu 网络带宽(内外网) 
第三方依赖 数据库 中间件 第三方api 额外服务(指纹识别问题) 连接状态 主动连接被动连接(容器开通随机端口ftp)


#### 容量规划 ####
监控当前服务 资源占用情况
对比docker服务


#### 价值 ####
建立资源池
资源利用率提高
快速部署构建



