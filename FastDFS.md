# FastDFS



七牛云 s3 存储

fastdfs 搭建

springboot restful api



## 文件上传服务（SpringBoot RESTful API）

```
http://fastdfs.biodwhu.cn/upload
```

（form表单上传） name="fastdfs"

app_id

app_key



请求  fastdfs





文件下载（文件预览）

```
http://fastdfs.biodwhu.cn/download/fileid

18.2.2.2:8888/key
```

## 







***高可用必须有崩溃恢复的能力，集群必须有数据同步的能力，不然部署多个服务器只是实现了负债均衡***



![1547453485452](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1547453485452.png)



## 单节点搭建



测试上传

交互式进入容器

```
docker exec -it fastdfs /bin/bash
```


## 测试文件上传


```
#拷贝服务器上文件到容器虚拟机内，fastdfs为容器名，冒号右边为容器内部地址
docker cp xxx.xx fastdfs:.

    /usr/bin/fdfs_upload_file /etc/fdfs/client.conf 
/usr/bin/fdfs_download_file /etc/fdfs/client.conf group1/M00/00/00/rBACFFyAiGCAUPM9AAACXGJhbms03.conf /filename


V2.05提供了比较正式的三个小工具：
      上传文件：/usr/bin/fdfs_upload_file  <config_file> <local_filename>
      下载文件：/usr/bin/fdfs_download_file <config_file> <file_id> [local_filename]
       删除文件：/usr/bin/fdfs_delete_file <config_file> <file_id>
```



### 服务器反馈上传地址



```
group1/M00/00/00/wKhLi1oHVMCAT2vrAAAeSwu9TgM3976771
```



## Group调试

​	将实验室三台存储器作为Storage Server，可以通过任意一台服务器查看当前Group内所有server的运行情况



- FDFS_STORAGE_STATUS：INIT      :初始化，尚未得到同步已有数据的源服务器

- FDFS_STORAGE_STATUS：WAIT_SYNC :等待同步，已得到同步已有数据的源服务器

- FDFS_STORAGE_STATUS：SYNCING   :同步中

- FDFS_STORAGE_STATUS：DELETED   :已删除，该服务器从本组中摘除

- FDFS_STORAGE_STATUS：OFFLINE   :离线

- FDFS_STORAGE_STATUS：ONLINE    :在线，尚不能提供服务

- FDFS_STORAGE_STATUS：ACTIVE    :在线，可以提供服务



  **通过以下命令查询storage状态**

```
    fdfs_fmonitor /etc/fdfs/client.conf
```

​	

```
    /usr/bin/fdfs_test /etc/fdfs/client.conf upload /usr/include/stdlib.h
```









在实际操作过程中，配置了三台storage服务器都放置在group1内，返回信息大致如下：



```

        Storage 1:
                id = 172.16.2.20
                ip_addr = 172.16.2.20  WAIT_SYNC

        Storage 2:
                id = 172.16.2.21
                ip_addr = 172.16.2.21  ACTIVE

        Storage 3:
                id = 172.16.2.22
                ip_addr = 172.16.2.22  ACTIVE
                
                
                
```



事实上，返回信息还包含了集群的可用容量，跟踪器等信息,其中空余容量为数台服务器中的空余容量的最小值。

```

group name = group1
disk total space = 51171 MB
disk free space = 44425 MB

```



1. 其中1号节点处于WAIT_SYNC状态，然而期间并没有大量文件需要同步，需排查原因。

2. 二三号节点均处于ACTIVE状态，但在二号节点上传的压缩文件包并没有同步到三号节点中，导致将二号服务暂时关闭后，不能通过文件的URL进行下载。fastdfs容器暂停命令：


```
cd /usr/local/docker/fastdfs
docker-compose down
```





## WAIT_SYNC状态

​	由于暂没有数据文件，先行删除该storage节点。

```

fdfs_monitor /etc/fdfs/client.conf delete group1 10.1.8.x
```

​	删除成功后，查看storage状态信息会发现为DELETED状态。

​	

​	重新加入storage节点，重启docker-compose服务

​	**无效：但出现了新的错误INFO信息，提示不能连接到另一存储服务器的23000端口，考虑防火墙的问题**



防火墙开启所有存储服务器的23000 TCP端口，并重启防火墙：

```
firewall-cmd --zone=public --add-port=23000/tcp --permanent
	
firewall-cmd --reload

firewall-cmd --list-ports
```

查看防火墙开放的端口



### **开放端口号后成功联通，且即使暂时中断两台存储服务器，变成OFFLINE状态，仍然可以下载文件，分布式搭建成功。**







### 





### 客户端上传错误日志一则



```java
 
ERROR - file: sockopt.c, line: 1682, max_count: 16 < iterface count: 16
[2019-02-25 06:23:46] 


INFO - fastdfs apache / nginx module v1.15, response_mode=proxy, base_path=/tmp, url_have_group_name=1, group_name=group1, storage_server_port=23000, path_count=1, store_path0=/fastdfs/storage, connect_timeout=10, network_timeout=30, tracker_server_count=1, if_alias_prefix=, local_host_ip_count=7, anti_steal_token=0, token_ttl=0s, anti_steal_secret_key length=0, token_check_fail content_type=, token_check_fail buff length=0, load_fdfs_parameters_from_tracker=1, storage_sync_file_max_delay=86400s, use_storage_id=0, storage server id count=0, flv_support=1, flv_extension=flv


2019/02/25 06:28:34 [error] 11#0: *1 connect() failed (113: No route to host) while connecting to upstream, client: 172.16.131.60, server: localhost, request: "GET /group1/M00/00/00/rBACFFxzimSAKQwnABlAeEmP4GA675.exe HTTP/1.1", upstream: "http://172.16.2.20:8888/group1/M00/00/00/rBACFFxzimSAKQwnABlAeEmP4GA675.exe?redirect=1", host: "172.16.2.23:8888"

```



**还是需要彻底关闭防火墙才能解决此问题**

```shell
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```





## 调试工作



上传相同图片出现以下Nginx错误



​	![1552030331860](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1552030331860.png)



Nginx错误日志文件有如下报告：



```nginx

root@localhost:/usr/local/nginx/logs# tail -f error.log

[2019-03-08 07:26:30] ERROR - file: /usr/local/src/fastdfs-nginx-module-master/src/common.c, line: 1093, file: /fastdfs/storage/dat0/rBACFlx_qbCAZsEPAADjtFeNj8w952.png not exist
2019/03/08 07:28:18 [error] 11#0: *308 connect() failed (113: No route to host) while connecting to upstream, client: 172.16.2.23, : localhost, request: "GET /group1/M00/00/00/rBACFlyCGYiAXKh2AABeCMxi0GM621.jpg HTTP/1.0", upstream: "http://172.16.2.22:8888/group00/00/rBACFlyCGYiAXKh2AABeCMxi0GM621.jpg?redirect=1", host: "fastdfs-cloud.biodwhu.cn"
2019/03/08 07:28:20 [error] 11#0: *310 connect() failed (113: No route to host) while connecting to upstream, client: 172.16.2.23, : localhost, request: "GET /group1/M00/00/00/rBACFlyCGYiAXKh2AABeCMxi0GM621.jpg HTTP/1.0", upstream: "http://172.16.2.22:8888/group00/00/rBACFlyCGYiAXKh2AABeCMxi0GM621.jpg?redirect=1", host: "fastdfs-cloud.biodwhu.cn"
2019/03/08 07:28:22 [error] 11#0: *312 connect() failed (113: No route to host) while connecting to upstream, client: 172.16.2.23, : localhost, request: "GET /group1/M00/00/00/rBACFlyCGYiAXKh2AABeCMxi0GM621.jpg HTTP/1.0", upstream: "http://172.16.2.22:8888/group00/00/rBACFlyCGYiAXKh2AABeCMxi0GM621.jpg?redirect=1", host: "fastdfs-cloud.biodwhu.cn"
2019/03/08 07:28:47 [error] 11#0: *318 connect() failed (113: No route to host) while connecting to upstream, client: 172.16.2.23, : localhost, request: "GET /group1/M00/00/00/rBACFlyCGYiAXKh2AABeCMxi0GM621.jpg HTTP/1.0", upstream: "http://172.16.2.22:8888/group00/00/rBACFlyCGYiAXKh2AABeCMxi0GM621.jpg?redirect=1", host: "fastdfs-cloud.biodwhu.cn"
2019/03/08 07:29:14 [error] 11#0: *320 connect() failed (113: No route to host) while connecting to upstream, client: 172.16.2.23, : localhost, request: "GET /group1/M00/00/00/rBACFlyCGb-AOo6UAABeCMxi0GM606.jpg HTTP/1.0", upstream: "http://172.16.2.22:8888/group00/00/rBACFlyCGb-AOo6UAABeCMxi0GM606.jpg?redirect=1", host: "fastdfs-cloud.biodwhu.cn"
2019/03/08 07:29:37 [emerg] 236#0: bind() to 0.0.0.0:8888 failed (98: Address already in use)
2019/03/08 07:29:37 [emerg] 236#0: bind() to 0.0.0.0:8888 failed (98: Address already in use)
2019/03/08 07:29:37 [emerg] 236#0: bind() to 0.0.0.0:8888 failed (98: Address already in use)
2019/03/08 07:29:37 [emerg] 236#0: bind() to 0.0.0.0:8888 failed (98: Address already in use)

```





第一次上传文件亦有此类错误：

```sql

2019/03/08 08:06:47 [error] 11#0: *337 connect() failed (113: No route to host) while connecting to upstream, client: 172.16.2.23, server: localhost, request: "GET /group1/M00/00/00/rBACFlyCIouABlwCAAgiQrBODhk382.pdf HTTP/1.0", upstream: "http://172.16.2.22:8888/group1/M00/00/00/rBACFlyCIouABlwCAAgiQrBODhk382.pdf?redirect=1", host: "fastdfs-cloud.biodwhu.cn"
```





```shell
        group_name=group1
        ip_addr=172.16.2.20
        status=7
        version=5.12
        join_time=1548246062
        storage_port=23000
        storage_http_port=8888
        domain_name=
        sync_src_server=
        sync_until_timestamp=0
        store_path_count=1
        subdir_count_per_path=256
        upload_priority=10
        total_mb=51171
        free_mb=44390
        total_upload_count=23
        success_upload_count=23
        total_append_count=0
        success_append_count=0
        total_set_meta_count=22
        success_set_meta_count=22
        total_delete_count=0
        success_delete_count=0
        total_download_count=0
        success_download_count=0
        total_get_meta_count=0
        success_get_meta_count=0
        total_create_link_count=0
        success_create_link_count=0
        total_delete_link_count=0
        success_delete_link_count=0
        total_upload_bytes=35208306
        success_upload_bytes=35208306
        total_append_bytes=0
        success_append_bytes=0
        total_download_bytes=0
        success_download_bytes=0
        total_sync_in_bytes=2454776
        success_sync_in_bytes=2454776
        total_sync_out_bytes=0
        success_sync_out_bytes=0
        total_file_open_count=33
        success_file_open_count=33
        total_file_read_count=0
        success_file_read_count=0
        total_file_write_count=171
        success_file_write_count=171
        last_source_update=1551245590
        last_sync_update=1551071582
        last_synced_timestamp=0
        last_heart_beat_time=1551246330
        changelog_offset=0

```





```shell
# thread stack size, should > 512KB
# default value is 1MB
thread_stack_size=1MB
# 线程栈的大小。FastDFS server端采用了线程方式。更正一下，tracker server线程栈不应小于64KB，不是512KB。
# 线程栈越大，一个线程占用的系统资源就越多。如果要启动更多的线程（V1.x对应的参数为max_connections，
V2.0为work_threads），可以适当降低本参数值。


//在sample中，配置文件该值为64KB，最小值为64kb，可以适当增加到512KB,或者1MB


```



疑点：



在tracker.conf配置文件中，http端口为8080端口，疑与8888不符

```shell
# HTTP port on this tracker server
http.server_port=8080

# check storage HTTP server alive interval seconds
# <= 0 for never check
# default value is 30
http.check_alive_interval=30

```



storage.conf



```shell
# the storage server port
port=23000
#  storage server服务端口

# connect timeout in seconds
# default value is 30s
connect_timeout=30
#连接超时时间，针对socket套接字函数connect

# network timeout in seconds
network_timeout=60
#  storage server 网络超时时间，单位为秒。发送或接收数据时，如果在超时时间后还不能发送或接收数据，则本次网络通信失败。


# the buff size to recv / send data
# default value is 64KB
# since V2.00
buff_size = 256KB
# V2.0引入本参数。设置队列结点的buffer大小。工作队列消耗的内存大小 = buff_size * max_connections
# 设置得大一些，系统整体性能会有所提升。
# 消耗的内存请不要超过系统物理内存大小。另外，对于32位系统，请注意使用到的内存不要超过3GB
```





可疑参数：

```shell
#仅接受一个线程？那工作线程设置为4则是为何？连接数与线程数有什么联系？
# accept thread count
# default value is 1
# since V4.07
accept_threads=1

```



```shell
# sync start time of a day, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
sync_start_time=00:00

# sync end time of a day, time format: Hour:Minute
# Hour from 0 to 23, Minute from 0 to 59
sync_end_time=23:59
# 上面二个一起解释。允许系统同步的时间段 (默认是全天) 。一般用于避免高峰同步产生一些问题而设定，相信sa都会明白
```





```shell

2019/03/12 08:42:27 [error] 10#0: *8 connect() failed (111: Connection refused) while connecting to upstream, client: 172.16.2.23, server: localhost, request: "GET /group1/M00/00/00/rBACFFyHS5aAfcBVAAAexZJI7KM453.png HTTP/1.0", upstream: "http://172.16.2.20:8888/group1/M00/00/00/rBACFFyHS5aAfcBVAAAexZJI7KM453.png?redirect=1", host: "fastdfs-cloud.biodwhu.cn"

```





**用fdfs_test进行上传测试，生成了详细的上传信息，发现图片实际访问的URL地址为存储服务器的IP开头，猜测通过调度服务器下载时Nginx进行了重定向操作。而“失效的”上传文件通过存储服务器组成的URL则可以访问到。**



```
/usr/bin/fdfs_test /etc/fdfs/client.conf upload /usr/include/stdlib.h
This is FastDFS client test program v5.12

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.
Please visit the FastDFS Home Page http://www.csource.org/
for more detail.

[2019-03-13 06:42:19] DEBUG - base_path=/fastdfs/tracker, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group:
        server 1. group_name=, ip_addr=172.16.2.21, port=23000

group_name=group1, ip_addr=172.16.2.21, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/rBACFVyIpkyAW-GPAACDlqtGkq042055.h
source ip address: 172.16.2.21
file timestamp=2019-03-13 06:42:20
file size=33686
file crc32=2873529005
example file url: http://172.16.2.21/group1/M00/00/00/rBACFVyIpkyAW-GPAACDlqtGkq042055.h
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/rBACFVyIpkyAW-GPAACDlqtGkq042055_big.h
source ip address: 172.16.2.21
file timestamp=2019-03-13 06:42:20
file size=33686
file crc32=2873529005
example file url: http://172.16.2.21/group1/M00/00/00/rBACFVyIpkyAW-GPAACDlqtGkq042055_big.h

```







# FastDFS为什么要结合Nginx？



### fastdfs-nginx-module模块失效，是否可用Nginx反向代理充当负载均衡功能？



我们在使用FastDFS部署一个分布式文件系统的时候，通过FastDFS的客户端API来进行文件的上传、下载、删除等操作。同时通过FastDFS的HTTP服务器来提供HTTP服务。但是FastDFS的HTTP服务较为简单，无法提供负载均衡等高性能的服务，所以FastDFS的开发者——淘宝的架构师余庆同学，为我们提供了Nginx上使用的FastDFS模块（也可以叫FastDFS的Nginx模块）。其使用非常简单。
FastDFS通过Tracker服务器,将文件放在Storage服务器存储,但是同组之间的服务器需要复制文件,有延迟的问题.假设Tracker服务器将文件上传到了192.168.1.80,文件ID已经返回客户端,这时,后台会将这个文件复制到192.168.1.30,如果复制没有完成,客户端就用这个ID在192.168.1.30取文件,肯定会出现错误。这个fastdfs-nginx-module可以重定向连接到源服务器取文件,避免客户端由于复制延迟的问题,出现错误。





Tracker server在内存中记录分组和Storage server的状态等信息，不记录文件索引信息，占用的内存量很少。Tracker server非常轻量化，不会成为系统瓶颈。



不存在fdfs_define.h?????

```
In file included from /usr/local/src/fastdfs-nginx-module-master/src/ngx_http_fastdfs_module.c:6:0:
/usr/local/src/fastdfs-nginx-module-master/src/common.c:26:33: fatal error: fastdfs/fdfs_define.h: No such file or directory
 #include "fastdfs/fdfs_define.h"

```









## 教程忽略操作记录



1.cd /usr/lib64

ll libfast*

cp libfastcommon.so /usr/lib



2. cd fastdfs/conf   复制完配置文件后，tracker服务才能启动！！！

   ll

   anti-steal.jpg

   client.conf

   storage_ids.conf

   

   cp * /etc/fdfs/



3. cd /usr/bin

fdfs_trackerd /etc/fdfs/tracker.conf

启动tracker





