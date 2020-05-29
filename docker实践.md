### docker实践 ###

#### 配置规划 ####
容器分区、系统内核、Docker版本、Docker Daemon访问控制及日志审计


容器分区
日志收集

#### tag ####
 version information, intended destination (prod or test, for instance), stability, or other information 


One case where it is appropriate to use bind mounts is during development, when you may want to mount your source directory or a binary you just built into your container. For production, use a volume instead, mounting it into the same location as you mounted a bind mount during development.