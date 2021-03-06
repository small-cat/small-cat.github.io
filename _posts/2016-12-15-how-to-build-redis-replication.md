---
layout: post
title: "redis 主备演练"
date: 2016-12-15 23:30:00
tags: redis 主备
---
搭建redis主从复制环境

redis 支持主从同步，数据可以从主服务器向任意数量的从服务器上同步，使用的是发布/订阅机制。

## 配置
在 github 上下载 redis 源码，在 linux 环境下编译，程序会在源目录中生成 `redis-server`，`redis-cli`，`redis-sentinel`等可执行文件。

LZ下载的是 redis-3.0.7。

### master 的配置
在源码根目录下，有一个 `redis.config` 配置文件，就是 redis 的配置文件，该配置文件中，对每一个配置参数都有详细的说明，读者可以自行查看。 (下面给出的是 master 的主要基本配置)

	# 使用守护进程模式
	daemonsize yes
	# 端口号
	port 6379
	# 绑定ip，本机的ip，LZ使用的多个虚拟机的方式，master 是 RHEL，slave 是 Ubuntu-server
	192.168.192.163
	# 学习开发，使用最大日志级别，能够看到最多的日志信息
	loglevel debug
	# 设定日志保存路径，需要指定日志输出的文件名。在非守护进程状态下，日志文件名为空，默认输出到stdout
	# 如果在守护进程模式下，日志文件名为空，那么默认输出到 /dev/null
	logfile "/home/XX/redis-3.0.7/log/redis-master.log"
	# 设置保存的 rdb 文件民
	dbfilename dump.rdb
	# rdb 文件保存路径
	dir /home/XX/redis-3.0.7/data/
	# 设置访问密码
	masterauth abc-1234
	# 客户端访问，需要密码连接
	# requirepass abc-1234
	
启动 master
	
	./redis-server redis-master.config
	
使用客户端连接，INFO 命令查看 master 的状态信息

	redis-cli -h 192.168.192.163 -p 6379 -a abc-1234
![status of redis master](http://oszgzpzz4.bkt.clouddn.com/image/redis_analysis/redis-master.PNG) <br>

### slave 的配置
slave 的配置与 master 的大体相同，不需要绑定ip，但是需要指定 master 的ip

	# 使用守护进程模式
	daemonsize yes
	# 端口号
	port 6380
	# 指定 master 的ip
	slaveof 192.168.192.163 6379
	# 学习开发，使用最大日志级别，能够看到最多的日志信息
	loglevel debug
	# 设定日志保存路径，需要指定日志输出的文件名。在非守护进程状态下，日志文件名为空，默认输出到stdout
	# 如果在守护进程模式下，日志文件名为空，那么默认输出到 /dev/null
	logfile "/home/XX/redis-3.0.7/log/redis-master.log"
	# 设置保存的 rdb 文件民
	dbfilename dump.rdb
	# rdb 文件保存路径
	dir /home/XX/redis-3.0.7/data/
	# 设置访问密码
	masterauth abc-1234
	# 客户端访问，需要密码连接
	# requirepass abc-1234
	
启动 slave，客户端登陆查看 `info replication`

![redis slave info replication](http://oszgzpzz4.bkt.clouddn.com/image/redis_analysis/redis-slave.PNG) <br>
查看日志，full sync 同步

	02:45:19.703 * Full resync from master: 35aa06b5e312e6c743dc6751ef5122bcb04775db:1
	02:45:20.992 * MASTER <-> SLAVE sync: receiving 53920275 bytes from master
	02:45:21.559 * MASTER <-> SLAVE sync: Flushing old data
	02:45:22.108 * MASTER <-> SLAVE sync: Loading DB in memory
	02:45:22.913 * MASTER <-> SLAVE sync: Finished with success

### HA监控 redis-sentinel
修改默认配置文件 sentinel.config

	# 指定监控的 master，当至少有一个 sentinel 认为 master 是 O_DOWN 状态时，master 为 failover
	sentinel monitor mymaster 192.168.192.163 6379 1
	# master 和 slave 的密码需要一致，也可以不设置密码
	sentinel auth-pass mymaster abc-1234
	
启动sentinel，`./redis-sentinel sentinel.config`，日志信息如下所示

	05:45:04.928 # Sentinel runid is 622e7db5c7589873430b40caa8a780051a097a89
	05:45:04.928 # +monitor master mymaster 192.168.192.163 6379 quorum 1
	05:45:05.929 * +slave slave 192.168.192.132:6380 192.168.192.132 6380 @ mymaster 192.168.192.163 6379
	05:45:05.930 * +slave slave 192.168.192.133:6381 192.168.192.133 6381 @ mymaster 192.168.192.163 6379
	
在配置文件中，不需要指定 slaves，sentinel 会自动监控到这些 slaves 并将信息回写到配置文件中，如上面的日志中最后两行显示的内容，就是 sentinel 监控到的 master 的 slaves，这些信息会回写到 sentinel.config 中

	# Generated by CONFIG REWRITE
	sentinel known-slave mymaster 192.168.192.133 6381
	sentinel known-slave mymaster 192.168.192.132 6380
	sentinel current-epoch 0 
	
info sentinel 查看信息

	# Sentinel
	sentinel_masters:1
	sentinel_tilt:0
	sentinel_running_scripts:0
	sentinel_scripts_queue_length:0
	master0:name=mymaster,status=ok,address=192.168.192.132:6380,slaves=2,sentinels=1
	
现在将 master SHUTDOWN，观察变化

	06:20:51.047 # +try-failover master mymaster 192.168.192.163 6379
	06:20:51.048 # +vote-for-leader 626c84dc1ce5849eca2bc4844c46faef70394e3b 1
	06:20:51.048 # +elected-leader master mymaster 192.168.192.163 6379
	06:20:51.049 # +failover-state-select-slave master mymaster 192.168.192.163 6379
	06:20:51.112 # +selected-slave slave 192.168.192.132:6380 192.168.192.132 6380 @ mymaster 192.168.192.163 6379
	06:20:51.112 * +failover-state-send-slaveof-noone slave 192.168.192.132:6380 192.168.192.132 6380 @ mymaster 192.168.192.163 6379
	06:20:51.213 * +failover-state-wait-promotion slave 192.168.192.132:6380 192.168.192.132 6380 @ mymaster 192.168.192.163 6379
	06:20:52.104 # +promoted-slave slave 192.168.192.132:6380 192.168.192.132 6380 @ mymaster 192.168.192.163 6379
	06:20:52.104 # +failover-state-reconf-slaves master mymaster 192.168.192.163 6379
	06:20:52.162 * +slave-reconf-sent slave 192.168.192.133:6381 192.168.192.133 6381 @ mymaster 192.168.192.163 6379
	06:20:53.145 * +slave-reconf-inprog slave 192.168.192.133:6381 192.168.192.133 6381 @ mymaster 192.168.192.163 6379
	06:20:55.751 * +slave-reconf-done slave 192.168.192.133:6381 192.168.192.133 6381 @ mymaster 192.168.192.163 6379
	06:20:55.842 # +failover-end master mymaster 192.168.192.163 6379
	06:20:55.842 # +switch-master mymaster 192.168.192.163 6379 192.168.192.132 6380
	06:20:55.842 * +slave slave 192.168.192.133:6381 192.168.192.133 6381 @ mymaster 192.168.192.132 6380
	06:20:55.843 * +slave slave 192.168.192.163:6379 192.168.192.163 6379 @ mymaster 192.168.192.132 6380
	06:21:25.854 # +sdown slave 192.168.192.163:6379 192.168.192.163 6379 @ mymaster 192.168.192.132 6380
	
从日志中看出，master(192.168.192.163) 被关闭之后，sentinel 将 192.168.192.132 提升为 master，同时将 192.168.192.133 和原来的master 192.168.192.163 作为 slave，但是 192.168.192.163 的状态为 sdown(subjectively down)，配置文件信息也会回写到 sentinel.config 中，如下所示

	# Generated by CONFIG REWRITE
	sentinel known-slave mymaster 192.168.192.163 6379
	sentinel known-slave mymaster 192.168.192.133 6381
	sentinel current-epoch 1
	
与刚开始启动时不同的是就是master 和 slave 变化了

此时查看 192.168.192.132 的状态 `info replication`

	# Replication
	role:master
	connected_slaves:1
	slave0:ip=192.168.192.133,port=6381,state=online,offset=6820,lag=0
	master_repl_offset:6820
	repl_backlog_active:1
	repl_backlog_size:1048576
	repl_backlog_first_byte_offset:2
	repl_backlog_histlen:6819
	
已经成为了 master，且连接的 slave 数量为1，如果此时，在将 192.168.192.163 启动呢，先查看 192.168.192.163 的状态

	role:slave
	master_host:192.168.192.132
	master_port:6380
	master_link_status:up
	master_last_io_seconds_ago:1
	master_sync_in_progress:0
	slave_repl_offset:50313
	slave_priority:100
	slave_read_only:1
	connected_slaves:0
	master_repl_offset:0
	repl_backlog_active:0
	repl_backlog_size:1048576
	repl_backlog_first_byte_offset:0
	repl_backlog_histlen:0
	
变成了 master，且 redis 会在配置文件 redis.config 回写一些配置，如下
	
	# Generated by CONFIG REWRITE
	slaveof 192.168.192.132 6380
	
回写的配置指定了 master 的 ip 和 端口号，同时可以通过 sentinel 的日志看到，将 192.168.192.163 转换成了 192.168.192.132 的 slave

	06:32:27.693 * +convert-to-slave slave 192.168.192.163:6379 192.168.192.163 6379 @ mymaster 192.168.192.132 6380
	
再来查看一下当前的 master 的状态

	# Replication
	role:master
	connected_slaves:2
	slave0:ip=192.168.192.133,port=6381,state=online,offset=74265,lag=0
	slave1:ip=192.168.192.163,port=6379,state=online,offset=74265,lag=0
	master_repl_offset:74265
	repl_backlog_active:1
	repl_backlog_size:1048576
	repl_backlog_first_byte_offset:2
	repl_backlog_histlen:74264
	
为启动 192.168.192.163 时，slave 的个数是 1 个，启动之后，变成了 2 个。
