# MySQL运行机制原理&架构

## 1. MySQL CS 逻辑架构

客户端支持的连接器 ：

Native C API; JDBC; ODBC; .NET; PHP; Perl; Python;Ruby; Cobol

服务端的运行逻辑结构：

服务管理程序

客服端程序命令：  `mysqladmin`、`mysqldump`、`mysqlcheck`等 

支持：备份，回滚，安全（登录、增删改查），拷贝等操作



连接池

主要用于多实例并发环境，具体包括权限，最大线程数，连接限制，缓存大小等，部分参数设置与mysql.cnf配置文件中，修改配置文件之后需要重启MySQL服务器，可以参考 [mysql.cnf参数]()具体说明：

系统支持的最大连接max_connections

用户能最大连接进来的数量max_user_connections



## mysql.cnf文件参数说明

```
#vim /etc/my.cnf 以下只列出my.cnf文件中[mysqld]段落中的内容
[mysqld]
port = 3306
#本地连接
serverid = 1
#套接字socket连接时自动生成的临时文件
socket = /tmp/mysql.sock
#避免MySQL的外部锁定
skip-locking
#禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求
skip-name-resolve
#back_log 参数的值指出在MySQL暂时停止响应新请求之前的短时间内多少个请求可以被存在堆栈中。 如果系统在一个短时间内有很多连接，则需要增大该参数的值，该参数值指定到来的TCP/IP连接的侦听队列的大小。不同的操作系统在这个队列大小上有它自 己的限制。 试图设定back_log高于你的操作系统的限制将是无效的。默认值为50。对于Linux系统推荐设置为小于512的整数
back_log = 384
#key_buffer_size指定用于索引的缓冲区大小，增加它可得到更好的索引处理性能。对于内存在4GB左右的服务器该参数可设置为256M或384M。注意：该参数值设置的过大反而会是服务器整体效率降低
key_buffer_size = 256M
#限制server接受的数据包的最大值，大的插入和更新会被max_allowed_packet参数限制,导致失败
max_allowed_packet = 4M
#每个连接线程被创建时的初始内存大小，不会自动调整，过小会影响执行增加修改的操作
thread_stack = 256K
#设置table高速缓存的数量，即当前缓存的table总占用的内存大小，参数值也与max_connections有关。
table_cache = 128K
sort_buffer_size = 6M
#查询排序时所能使用的缓冲区大小。注意：该参数对应的分配内存是每连接独占，如果有100个连接，那么实际分配的总共排序缓冲区大小为100 × 6 ＝ 600MB。所以，对于内存在4GB左右的服务器推荐设置为6-8M。
read_buffer_size = 4M
#读查询操作所能使用的缓冲区大小。和sort_buffer_size一样，该参数对应的分配内存也是每连接独享。
join_buffer_size = 8M
#联合查询操作所能使用的缓冲区大小，和sort_buffer_size一样，该参数对应的分配内存也是每连接独享。
myisam_sort_buffer_size = 64M
table_cache = 512
thread_cache_size = 64
query_cache_size = 64M
# 指定MySQL查询缓冲区的大小。可以通过在MySQL控制台观察，如果Qcache_lowmem_prunes的值非常大，则表明经常出现缓冲不够的 情况；如果Qcache_hits的值非常大，则表明查询缓冲使用非常频繁，如果该值较小反而会影响效率，那么可以考虑不用查询缓 冲；Qcache_free_blocks，如果该值非常大，则表明缓冲区中碎片很多。
tmp_table_size = 256M
max_connections = 768
#指定MySQL允许的最大连接进程数。如果在访问论坛时经常出现Too Many Connections的错误提 示，则需要增大该参数值。
max_connect_errors = 10000000
wait_timeout = 10
#指定一个请求的最大连接时间，对于4GB左右内存的服务器可以设置为5-10。
thread_concurrency = 8
#该参数取值为服务器逻辑CPU数量*2，在本例中，服务器有2颗物理CPU，而每颗物理CPU又支持H.T超线程，所以实际取值为4*2=8
skip-networking
#开启该选项可以彻底关闭MySQL的TCP/IP连接方式，如果WEB服务器是以远程连接的方式访问MySQL数据库服务器则不要开启该选项！否则将无法正常连接！
table_cache=1024
#物理内存越大,设置就越大.默认为2402,调到512-1024最佳
innodb_additional_mem_pool_size=4M
#默认为2M
innodb_flush_log_at_trx_commit=1
#设置为0就是等到innodb_log_buffer_size列队满后再统一储存,默认为1
innodb_log_buffer_size=2M
#默认为1M
innodb_thread_concurrency=8
#你的服务器CPU有几个就设置为几,建议用默认一般为8
key_buffer_size=256M
#默认为218，调到128最佳
tmp_table_size=64M
#默认为16M，调到64-256最挂
read_buffer_size=4M
#默认为64K
read_rnd_buffer_size=16M
#默认为256K
sort_buffer_size=32M
#默认为256K
thread_cache_size=120
#默认为60
query_cache_size=32M
```