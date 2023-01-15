# 西安项目MySQL调优记录 

## 问题描述
测试描述：开启多线程插入数据，服务端中止连接，日志原因未知   

## 猜测

### 最大连接数
配置为`max_connections = 3000` 首先想到最大连接数，修改为6000,猜测原因不在此    

同时使用MySQL命令查看`show global status like 'conn%';`

```bash

root@localhost> show global status like 'conn%';
+-----------------------------------+-------+
| Variable_name                     | Value |
+-----------------------------------+-------+
| Connection_errors_accept          | 0     |
| Connection_errors_internal        | 0     |
| Connection_errors_max_connections | 0     |
| Connection_errors_peer_address    | 0     |
| Connection_errors_select          | 0     |
| Connection_errors_tcpwrap         | 0     |
| Connections                       | 3216  |
+-----------------------------------+-------+
7 rows in set (0.00 sec)

```

可以看到MySQL连接`Connection_errors_max_connections`为0 说明没有到达最大连接数  


### 错误日志error.log
查看MySQL错误日志`error.log`，进行进一步分析，错误日志位于配置项

```cnf
[mysqld]
log-error=/home/mysql/data/mysqldata1/log/error.log
```

首先发现mysql重启前会强制断开很多的连接，日志表现为 `Forcing close of thread 1105740  user: 'zhcyusr'` 怀疑程序连接数据库后没有正常关闭（测试使用多线程批量创建数据）   

![](https://hexoric-1310528773.cos.ap-beijing.myqcloud.com/hexo/msyql西安调优1.png ':size=70%')  


其次发现启动后数据库警告`Got an error reading communication packets` 和`Aborted connection 2957 to db: 'zhcydb' user: 'zhcyusr' host: '111.111.111.11' (Got an error reading communication packets)`  

针对第一个警告：    
首先关注`Aborted_clients` `Aborted_connects` 两个变量值，命令`show global status like 'abort%';`进行查看

```bash
> show global status like 'abort%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| Aborted_clients  | 10    |
| Aborted_connects | 1311  |
+------------------+-------+
2 rows in set (0.00 sec)
```

`Aborted_clients`中止客户端，可能的原因有：   
- 客户端试图访问数据库，但是没有数据库权限
- 客户端使用了错误的密码
- 连接包含不正确的信息
- 获取一个连接包需要的时间超过`connect_timeout`秒

`Aborted_connects` 中止连接，可能的原因有：  
- 程序退出前没有调用`mysql_close()`关闭连接
- 客户端睡眠时间超过了`wait_timeout`或`interactive_timeout`参数的秒数
- 客户端程序在数据传输过程中突然中止

> 简单来说即：数据库会话未能正常连接到数据库，会造成`Aborted_connects`变量增加。数据库会话已正常连接到数据库但未能正常退出，会造成`Aborted_clients`变量增加    
> 会话异常退出一般会造成`Aborted connection`告警，即我们可以通过`Aborted_clients`状态变量的变化来反映出是否存在异常会话，那么出现`Got an error reading communication packets`类似告警的原因就很明了了，查询相关资料，总结出造成Aborted connection告警的可能原因如下：

- 会话链接未正常关闭，程序没有调用`mysql_close()`。
- 睡眠时间超过`wait_timeout`或`interactive_timeout`参数的秒数。
- 查询数据包大小超过`max_allowed_packet`数值，造成连接中断。
- 其他网络或者硬件层面的问题


`Aborted connection`告警是很难避免的，error log里或多或少会有少量Aborted connection信息，这种情况是可以忽略的    
建议：     
- 建议业务操作结束后，应用程序逻辑会正确关闭连接，以短连接替代长连接。
- 检查以确保max_allowed_packet的值足够高，并且客户端没有收到“数据包太大”消息`set global max_allowed_packet=1024;`。
- 确保客户端应用程序不中止连接，例如，如果PHP设置了max_execution_time为5秒，增加connect_timeout并不会起到作用，因为PHP会kill脚本。其他程序语言和环境也有类似的安全选项。
- 确保事务提交（begin和commit）都正确提交以保证一旦应用程序完成以后留下的连接是处于干净的状态。
- 检查是否启用了skip-name-resolve，检查主机根据其IP地址而不是其主机名进行身份验证。
- 尝试增加MySQL的net_read_timeout和net_write_timeout值，看看是否减少了错误的数量。



### 超时参数  

命令`show variables like '%timeout%';`

```bash
zhcyusr@localhost : (none) 10:44:21> show variables like '%timeout%';
+------------------------------+----------+
| Variable_name                | Value    |
+------------------------------+----------+
| connect_timeout              | 10       |
| delayed_insert_timeout       | 300      |
| have_statement_timeout       | YES      |
| innodb_flush_log_at_timeout  | 1        |
| innodb_lock_wait_timeout     | 120      |
| innodb_rollback_on_timeout   | ON       |
| interactive_timeout          | 172800   |
| lock_wait_timeout            | 31536000 |
| net_read_timeout             | 30       |
| net_write_timeout            | 60       |
| rpl_semi_sync_master_timeout | 1000     |
| rpl_stop_slave_timeout       | 31536000 |
| slave_net_timeout            | 10       |
| wait_timeout                 | 172800   |
+------------------------------+----------+
14 rows in set (0.00 sec)
```

`connect_timeout`在握手认证阶段（authenticate）起作用    
`interactive_timeout` 和wait_timeout在连接空闲阶段（sleep）起作用   
`net_read_timeout`和`net_write_timeout`则是在连接繁忙阶段（query）或者网络出现问题时起作用     

设置超时参数 `SET GLOBAL connect_timeout = 43200;`





# 参考
> - [简书](https://www.jianshu.com/p/827a973bb047)
> - [CSDN](https://blog.csdn.net/dreamyuzhou/article/details/117470191)