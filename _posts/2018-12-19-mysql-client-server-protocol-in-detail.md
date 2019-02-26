---
layout: post
title: 'MySQL Client/Server Protocol in Detail'
description: "MySQL数据库客户端服务端协议详述"
author: Marty Pang
categories: 
  - Database
tags: 
  - MySQL
  - Client
  - TCP/IP
  - Protocol
  - Mysql协议
  - 客户端
  - 服务端
  - 通信
---

# MySQL Client/Server Protocol in Detail
最近在做的一个系统项目需要支持MySQL客户端，遂研究了MySQL C/S 协议。MySQL实现了四种通信协议，包括：
- TCP/IP
- Unix Socket协议
- 命名管道
- 共享内存
  四种方式。其中后两种只支持在windows系统，Unix Socket由于受限于物理文件，只支持本地的访问。所以这里我们重点解析MySQL实现的基于TCP/IP连接的通信协议。

## 协议模型结构
下图给出MySQL通信协议从最上层应用层到底层数据传输层的模型结构：
![](/images/20181219/network_model.png){:  .align-center}

从上至下分别是：
- 应用层：客户端应用程序和服务端应用程序
- 中间层：NET层分别打包应用程序数据
- I/O层：Virtual IO层，是对network io的封装
- 数据传输层：数据传输采用Socket编程实现

## 通用报文的结构
所有的包都有统一的格式，并通过my_net_write()函数写入buffer等待发送。

```c++
+-------------------+--------------+---------------------------------------------------+
|      3 Bytes      |    1 Byte    |                   N Bytes                         |
+-------------------+--------------+---------------------------------------------------+
|<= length of msg =>|<= sequence =>|<==================== data =======================>|
|<============= header ===========>|<==================== body =======================>|
```

MySQL 报文格式如上，消息头包含了 A) 报文长度，标记当前请求的实际数据长度，以字节为单位；B) 序号，为了保证交互时报文的顺序，每次客户端发起请求时，序号值都会从 0 开始计算。
消息体用于存放报文的具体数据，长度由消息头中的长度值决定。
单个报文的最大长度为 (2^24-1) Bytes，大小为 (2^24-1) Bytes的包视作不完整的包，会与后面接收的包合并为一个，直到接收到一个长度小于 (2^24-1) Bytes的包。

## 认证协议
MySQL客户端与服务端的连接采用TCP的三次握手协议。在三次握手建立连接后，客户端与服务端进行认证过程。MySQL认证采用经典的CHAP协议，即挑战握手认证协议。简单介绍CHAP协议的执行过程：
1. 服务端发送随机字符串（握手初始化包）给客户端
2. 客户端对明文密码和随机字符串做加密，将得到的token（登录认证包）发送给服务端
3. 服务端验证token，发送给客户端验证结果包

客户端作如下计算：
```c++
token = SHA1(scramble + SHA1(SHA1(password))) XOR SHA1(password)         
      = PASSWORD XOR SHA1(password)
```

服务端作如下验证：
```c++
stage1_hash = token XOR SHA1(scramble + mysql.user.password) = token XOR PASSWORD
            = [PASSWORD XOR SHA1(password)] XOR PASSWORD
            = SHA1(password)
```

## 报文结构 服务端 --> 客户端

###a. 握手初始化包
MySQL握手初始化包的消息体结构如下图：
![](/images/20181219/handshake.png?raw=true){:  .align-center}

- 协议版本号：由宏PROTOCOL_VERSION定义（mysql源码/include/mysql_version.h）
- 服务器版本信息：由宏MYSQL_SERVER_VERSION定义（mysql源码/include/mysql_version.h）
- 服务器线程ID：当前连接创建线程的ID
- 挑战随机数：服务器生成的随机字符串，由create_random_scramble()函数生成
- 服务器权能标志：用于与客户端协商通讯方式，各标志位含义参考[链接](https://dev.mysql.com/doc/dev/mysql-server/latest/group__group__cs__capabilities__flags.html)（mysql源码/include/mysql_com.h）

###b. 结果包 （待补充）

## 报文结构 客户端 --> 服务端

###a. auth包
客户端登录认证包的消息体结构如下图：
![](/images/20181219/auth.png?raw=true){:  .align-center}

###b. command包
客户端命令请求包的消息体结构如下图：
![](/images/20181219/command.png?raw=true){:  .align-center}

- 命令：用来标志当前请求消息的类型。命令的取值范围及说明如下表所示（mysql源码/include/my_command.h）：

| 类型值  | 命令                      | 功能            | 关联函数                      |
| ---- | ----------------------- | ------------- | ------------------------- |
| 0x00 | COM_SLEEP               | （内部线程状态）      | （无）                       |
| 0x01 | COM_QUIT                | 关闭连接          | mysql_close               |
| 0x02 | COM_INIT_DB             | 切换数据库         | mysql_select_db           |
| 0x03 | COM_QUERY               | SQL查询请求       | mysql_real_query          |
| 0x04 | COM_FIELD_LIST          | 获取数据表字段信息     | mysql_list_fields         |
| 0x05 | COM_CREATE_DB           | 创建数据库         | mysql_create_db           |
| 0x06 | COM_DROP_DB             | 删除数据库         | mysql_drop_db             |
| 0x07 | COM_REFRESH             | 清除缓存          | mysql_refresh             |
| 0x08 | COM_SHUTDOWN            | 停止服务器         | mysql_shutdown            |
| 0x09 | COM_STATISTICS          | 获取服务器统计信息     | mysql_stat                |
| 0x0A | COM_PROCESS_INFO        | 获取当前连接的列表     | mysql_list_processes      |
| 0x0B | COM_CONNECT             | （内部线程状态）      | （无）                       |
| 0x0C | COM_PROCESS_KILL        | 中断某个连接        | mysql_kill                |
| 0x0D | COM_DEBUG               | 保存服务器调试信息     | mysql_dump_debug_info     |
| 0x0E | COM_PING                | 测试连通性         | mysql_ping                |
| 0x0F | COM_TIME                | （内部线程状态）      | （无）                       |
| 0x10 | COM_DELAYED_INSERT      | （内部线程状态）      | （无）                       |
| 0x11 | COM_CHANGE_USER         | 重新登陆（不断连接）    | mysql_change_user         |
| 0x12 | COM_BINLOG_DUMP         | 获取二进制日志信息     | （无）                       |
| 0x13 | COM_TABLE_DUMP          | 获取数据表结构信息     | （无）                       |
| 0x14 | COM_CONNECT_OUT         | （内部线程状态）      | （无）                       |
| 0x15 | COM_REGISTER_SLAVE      | 从服务器向主服务器进行注册 | （无）                       |
| 0x16 | COM_STMT_PREPARE        | 预处理SQL语句      | mysql_stmt_prepare        |
| 0x17 | COM_STMT_EXECUTE        | 执行预处理语句       | mysql_stmt_execute        |
| 0x18 | COM_STMT_SEND_LONG_DATA | 发送BLOB类型的数据   | mysql_stmt_send_long_data |
| 0x19 | COM_STMT_CLOSE          | 销毁预处理语句       | mysql_stmt_close          |
| 0x1A | COM_STMT_RESET          | 清除预处理语句参数缓存   | mysql_stmt_reset          |
| 0x1B | COM_SET_OPTION          | 设置语句选项        | mysql_set_server_option   |
| 0x1C | COM_STMT_FETCH          | 获取预处理语句的执行结果  | mysql_stmt_fetch          |

- 参数：用户或应用程序输入的内容，如SQL语句，调用存储过程等

## Client Server详细交互过程（code level）
```c++
### Client(mysql)  ###                       ### Server(mysqld) ###
----------------------------------------     --------------------------------------------------
main()                                       mysqld_main()
 |-sql_connect()                              |-init_ssl()
 | |-sql_real_connect() {for(;;)}             |-network_init()
 |   |-mysql_init()                           |-handle_connections_sockets()
 |   |-init_connection_options()                |-select()/poll()
 |   |-mysql_real_connect()                     |
 |     |-cli_mysql_real_connect()               |
 |       |-socket()                             |
 |       |-vio_new()                            |
 |       |-vio_socket_connect()                 |
 |       | |-mysql_socket_connect()             |
 |       |   |-connect()                        |
 |       |   |                                  |
 |       |   |        [Socket Connect]          |
 |       |   |>>==========>>==========>>======>>|
 |       |                                      |-accept()
 |       |-vio_keepalive()                      |-vio_new()
 |       |-my_net_set_read_timeout()            |-my_net_init()
 |       |-my_net_set_write_timeout()           |-create_new_thread()
 |       |-vio_io_wait()                          |-handle_one_connection()    {新线程}
 |       |                                          |-thd_prepare_connection() {for(;;)}
 |       |                                          | |-login_connection()
 |       |                                          |   |-check_connection()
 |       |                                          |     |-acl_check_host()
 |       |                                          |     |-vio_keepalive()
 |       |                                          |     |-acl_authenticate()
 |       |                                          |       |-do_auth_once()
 |       |                                          |       | |-native_password_authenticate()  {插件实现}
 |       |                                          |       |   |-create_random_string()
 |       |                                          |       |   |-send_server_handshake_packet()
 |       |                                          |       |   |
 |       |              [Handshake Initialization]  |       |   |
 |       |<<==<<==========<<==========<<==========<<==========<<|
 |       |-cli_safe_read()                          |       |   |-my_net_read()
 |       |-run_plugin_auth()                        |       |   |
 |       | |-native_password_auth_client()          |       |   |
 |       |   |-scramble()                           |       |   |
 |       |     |-my_net_write()                     |       |   |
 |       |     |                                    |       |   |
 |       |     |            [Client Authentication] |       |   |
 |       |     |>>==========>>==========>>==========>>========>>|
 |       |                                          |       |   |-check_scramble()
 |       |                                          |       |-mysql_change_db()
 |       |                                          |       |-my_ok()
 |       |                      [OK]                |       |
 |       |<<==========<<==========<<==========<<==========<<|
 |       |-cli_safe_read()                          |
 |                                                  |
 |                                                  |
 |                                                  |
 |                                                  |
 |-put_info() {Welcome Info}                        |
 |-read_and_execute() [for(;;)]                     |
                                                    |-thd_is_connection_alive()  [while()]
                                                    |-do_command()
```

## 参考文章
- [Chapter 14 MySQL Client/Server Protocol](https://dev.mysql.com/doc/internals/en/client-server-protocol.html)
- [MySQL 通讯协议](https://jin-yang.github.io/post/mysql-protocol.html)
- [Mysql通讯协议分析](http://codingo.xyz/index.php/2017/12/27/mysql_protocol/)


