title: 判断Redis复制是否完成的方法
date: 2014-09-25 15:44:24
categories: Redis 
tags: [redis, replication]
---

当需要使用Redis的复制功能时，有时需要能及时的得到复制完成的信息，或者说复制的进度。

<!--more-->

## 实现方法

### 方式1-可行

Redis提供的INFO命令，可以提供redis运行时的各种信息。我们这里需要关注Replication段：

redis-cli下连接到redis master服务器，执行info命令

	$ redis-cli
	127.0.0.1:6379> info Replication

得到以下输出：

	# Replication
	role:master
	connected_slaves:1
	slave0:ip=127.0.0.1,port=6371,state=online,offset=3132,lag=0
	master_repl_offset:3132
	repl_backlog_active:1
	repl_backlog_size:1048576
	repl_backlog_first_byte_offset:2
	repl_backlog_histlen:3131

role:master表示这是一个master，master_repl_offset:3132 可以得到当前master记录的复制偏移量。而"slave0:ip=127.0.0.1,port=6371,state=online,offset=3132,lag=0"是当前连接到这个master的slave的信息，offset=3132 是当前slave的复制偏移量。

在这个例子中，master和slave的offset相同，说明此刻slave已经复制到最新的数据。

总结，我们只需要在master端执行info replication命令，根据上面的输出，比较当前master和slave的offset，就可以知道当前slave是否同步完成。


### 方式2-不可行

类似的，如果在客户端执行info Replication命令：

	# Replication
	role:slave
	master_host:127.0.0.1
	master_port:6379
	master_link_status:up
	master_last_io_seconds_ago:7
	master_sync_in_progress:0
	slave_repl_offset:3132
	slave_priority:100
	slave_read_only:1
	connected_slaves:0
	master_repl_offset:0
	repl_backlog_active:0
	repl_backlog_size:1048576
	repl_backlog_first_byte_offset:0
	repl_backlog_histlen:0

这里同样可以通过slave_repl_offset得到当前slave的复制偏移量。  但是这里的master_repl_offset 字段，经过测试发现它的值始终为0，未能如名字所示的展示master此刻的offset。

这个不知道是否算是bug？或者这个master_repl_offset有其他特殊含义？——需要进一步查看。


