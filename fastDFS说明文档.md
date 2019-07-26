## 文件上传服务



实验室部署的文件存储服务利用开源的FastDFS框架构建而成，整体架构分为一台调度服务器与三台存储服务器，利用docker技术对服务进行部署。





## 服务端



#### 文件服务目录结构

- Fastdfs
  - environment 存放服务所需的压缩包，配置文件及dockerFile文件
  - storage/tracker  数据卷的挂载目录，存放服务运行中产生的日志文件与数据文件
  - docker-compose  docker服务启动文件



#### 全局启动方式：



调度服务节点与存储服务节点的启动方式类似，**均通过docker-compose方式进行启动**，docker容器的端口映射方式为桥接方式（host），数据卷挂载在../fastdfs/storage上



```
cd ../fastdfs
docker-compose up -d
```



#### 运行环境



##### 文件依赖模块：

- fastdfs-master.tar.gz  Fastdfs主模块
- fastdfs-nginx-module-master.tar.gz  Fastdfs-nginx中间模块
- libfastcommon.tar.gz  公共函数库
- nginx-1.15.1.tar.gz  NGINX服务



##### 服务配置文件：



- tracker.conf  调度服务器节点配置文件
- storage.conf  存储服务器节点配置文件 
- client.conf  客户端配置文件
- mod_fastdfs.conf  分布式存储系统存储高级设置（最大同步延迟时间、文件组名设置、日志级别等
- nginx.conf  NGINX Http方式访问配置选项，需将对指定端口的访问交给ngx_fastdfs_module模块进行处理



##### 启动服务文件：



- dockerFile  安装依赖包，FastDfs，NGINX，进行配置操作。

- ##### entrypoint.sh  在DockerFile中对模块进行完初始化以及配置好文件好，对容器中服务进行启动的命令,以下代码仅在本节点启动了tracker服务与nignx服务

  ```
  #!/bin/sh
  /etc/init.d/fdfs_trackerd start
  /usr/local/nginx/sbin/nginx -g 'daemon off;'
  ```

  



**若需在新服务器上搭建文件存储服务，需修改特定配置文件：**



```
storage.conf:

	tracker_server=新调度服务器IP地址:22122
	
client.conf

	tracker_server=新调度服务器IP地址:22122
	
mod_fastdfs.conf

	tracker_server=新调度服务器IP地址:22122
	
	
```



若当前节点用作调度节点：

则 entrypoint.sh 内容为

```
#!/bin/sh
/etc/init.d/fdfs_trackerd start
/usr/local/nginx/sbin/nginx -g 'daemon off;'
```



且DockerFile中必须进行以下配置操作

```
# 配置 FastDFS 存储
ADD storage.conf /etc/fdfs
RUN mkdir -p /fastdfs/storage
```



若为存储节点，entrypoint.sh内容为：

```
#!/bin/sh
/etc/init.d/fdfs_storaged start
/usr/local/nginx/sbin/nginx -g 'daemon off;
```



且DockerFile中必须进行以下配置操作

```
# 配置 FastDFS 跟踪器
ADD tracker.conf /etc/fdfs
RUN mkdir -p /fastdfs/tracker
```







## 客户端



#### 客户端接口文档:

​	

客户端apidoc文档可见：

[api文档](<http://fastdfs.biodwhu.cn/apidoc>)



FastDFS存储服务对后台开发开放文件上传，下载的接口。



为避免文件在后台进行多次传输导致流量增大，当客户端上传文件时，**需首先通过后台向FastDFS请求token权限，再将生成的jwt token返回给用户**，用户通过token与FastDFS直接通信，进行上传。后台仅起到权限验证与中转验证信息的作用。



后台为用户请求token时，需提供以下三个参数：

- appsecret  为服务用户生成的秘钥值，**与会过期的token不同，秘钥值可重复使用**
- user_name  各自平台下上传用户的用户名
- prefix 生成文件UID前缀的指定，若传递PF，则可返回”PF-190213900“



![1559650223701](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1559650223701.png)