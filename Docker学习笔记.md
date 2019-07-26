# Docker学习笔记



## Docker引擎

Docker引擎是一个包含一下主要组件的客户端服务器应用程序



- 一种服务器，它是一种称为守护进程且长时间运行的程序
- REST API用于指定程序可以用来与守护进程通信的接口，并指示它做什么
- 一个有命令行界面工具的客户端





## Docker架构

 

Docker使用C/S架构模式，使用API来管理和创建Docker容器



Docker容器通过Docker镜像来创建



容器与镜像的关系类似于面向对象编程中的对象与类

镜像之间还可以存在继承关系，比如Tomcat镜像继承java镜像



| Docker | 面向对象 |
| ------ | -------- |
| 容器   | 对象     |
| 镜像   | 类       |



容器不应该向其存储层写入任何数据，容器存储层要保持无状态化。所有的文件写操作，都应该使用**数据卷（Volume)**、或者**绑定宿主目录。**避免虚拟机中的重复写操作





## Docker仓库



镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其他服务器上使用这个镜像，需要一个集中的存储、分发镜像的服务。

Docker Registy就是这样的服务





## 安装Docker



### 准备工作

​	Docker CE  社区版  免费版

​	Docker EE  企业版  收费版



Docker CE可以安装在64位的x86平台或ARM平台上

LTS（Long-term Support)  长期维护，稳定版本



### 使用脚本自动安装



curl -fsSL get.docker.com -o get-docker.sh

sh get-docker.sh --mirror Aliyun



## Docker镜像加速器



国内从Docker Hub拉取镜像时会遇到困难，此时可以配置镜像加速器





## 镜像拉取与启动



docker pull [选项]  【Docker Registry 地址[:端口号]/】仓库名[:标签]

docker pull tomcat:jre-9  拉取镜像仓库中的镜像



docker  run  -p  8080:8080  tomcat     在8080端口启动镜像Tomcat



**查看镜像：docker images**

**查看运行中的容器： docker ps**

**查看所有容器：docker ps  -a**

**删除容器：docker rm  container-id**

**删除本地镜像：docker image rm image-id  /  docker rmi image-id**

**删除虚悬镜像： docker image purne**

 



## CentOS/RHEL 的用户需要注意的事项



在 Ubuntu/Debian 上有 `UnionFS` 可以使用，如 `aufs` 或者 `overlay2`，而 CentOS 和 RHEL 的内核中没有相关驱动。因此对于这类系统，一般使用 `devicemapper` 驱动利用 LVM 的一些机制来模拟分层存储。这样的做法除了性能比较差外，稳定性一般也不好，而且配置相对复杂。Docker 安装在 CentOS/RHEL 上后，会默认选择 `devicemapper`，但是为了简化配置，其 `devicemapper` 是跑在一个稀疏文件模拟的块设备上，也被称为 `loop-lvm`。这样的选择是因为不需要额外配置就可以运行 Docker，这是自动配置唯一能做到的事情。但是 `loop-lvm` 的做法非常不好，其稳定性、性能更差，无论是日志还是 `docker info` 中都会看到警告信息。官方文档有明确的文章讲解了如何配置块设备给 `devicemapper` 驱动做存储层的做法，这类做法也被称为配置 `direct-lvm`。

除了前面说到的问题外，`devicemapper` + `loop-lvm` 还有一个缺陷，因为它是稀疏文件，所以它会不断增长。用户在使用过程中会注意到 `/var/lib/docker/devicemapper/devicemapper/data` 不断增长，而且无法控制。很多人会希望删除镜像或者可以解决这个问题，结果发现效果并不明显。原因就是这个稀疏文件的空间释放后基本不进行垃圾回收的问题。因此往往会出现即使删除了文件内容，空间却无法回收，随着使用这个稀疏文件一直在不断增长。

所以对于 CentOS/RHEL 的用户来说，在没有办法使用 `UnionFS` 的情况下，一定要配置 `direct-lvm` 给 `devicemapper`，无论是为了性能、稳定性还是空间利用率。

*或许有人注意到了 CentOS 7 中存在被 backports 回来的 overlay 驱动，不过 CentOS 里的这个驱动达不到生产环境使用的稳定程度，所以不推荐使用。*





## 使用dockerfile定制镜像



Dockerfile 是一个文本文件，其内包含了一条条的**指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。



### **修改已存在的镜像配置**



要区分好几个概念：

- 交互的方式进入容器  docker exec -it  container-id bash
- 交互的方式启动容器  docker run -it tomcat bash
- 启动Tomcat  docker run -it  -p 8080:8080 tomcat



tomcat中，只能执行部分Linux命令，如：

- echo
- cat
- ls -la



在/var/local/tomcat中存放着容器文件









```
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

这个 Dockerfile 很简单，一共就两行。涉及到了两条指令，`FROM` 和 `RUN`

## FROM 指定基础镜像



FROM表示引用的基类镜像，在基类的基础上进行定制化



```
FROM tomcat
RUN echo "hello docker!" > /usr/local/tomcat/webapps/ROOT/index.html
```



在删除基类镜像不需要的ROOT文件夹中的文件时，不能直接用cd 切换到ROOT文件夹再删除所有，因为若如此，新创建的index.html将找不到ROOT文件夹，应该用WORKDIR 切换工作目录再删除

```
FROM tomcat
WORKDIR /usr/local/tomcat/webapps/ROOT/
RUN rm -fr *
RUN echo "hello docker!" > /usr/local/tomcat/webapps/ROOT/index.html
```



构建新镜像的指令：(该命令需要指定 "  .  " 当前目录下必须包含Dockerfile文件)

-t表示指定标签名

docker build -t myshop .







## 镜像构建的上下文



进行镜像构建的时候，并非所有定制都会通过 `RUN` 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 `COPY` 指令、`ADD` 指令等。而 `docker build` 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？



这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，`docker build` 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。****



```dockerfile
COPY ./package.json /app/
```

上面命令包含两个路径，第一个是上下文路径（相对路径）里的package.json，拷贝到绝对路径/app（通常拷贝到镜像的某个位置）



这并不是要复制执行 `docker build` 命令所在的目录下的 `package.json`，也不是复制 `Dockerfile` 所在目录下的 `package.json`，而是复制 **上下文（context）** 目录下的 `package.json`。

因此，`COPY` **这类指令中的源文件的路径都是*相对路径***。这也是初学者经常会问的为什么 `COPY ../package.json /app` 或者 `COPY /opt/xxxx /app` 无法工作的原因，因为这些路径**已经超出了上下文的范围，Docker 引擎无法获得这些位置的文件。**如果真的需要那些文件，应该将它们复制到上下文目录中去。





## DockerFile指令详解



已经介绍了FROM , RUN ,COPY 



目标：部署项目到docker



- 将myshop.zip的工程文件拷贝到准备构建镜像的上下文中（FTP方式）

- ```
  COPY my-shop.zip /usr/local/tomcat/webapps/ROOT
  ```

```
RUN unzip my-shop.zip   //这条命令将出错，因为“我”在工作目录（默认的工作目录为镜像的根目录，即/usr/local/tomcat），而要需要解压的文件在镜像的ROOT文件夹中
```

改成

```
WORKDIR /usr/local/tomcat/webapps/ROOT
COPY my-shop.zip .
RUN unzip my-shop.zip
```

- 修改端口号为80

  <!--在传统的项目部署中，通过配置server.xml配置端口号，通过bin目录下的startup.sh启动Tomcat-->



### WORKDIR 指定工作目录

使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。



一些初学者常犯的错误是把 `Dockerfile` 等同于 Shell 脚本来书写，这种错误的理解还可能会导致出现下面这样的错误：

```docker
RUN cd /app
RUN echo "hello" > world.txt
```



**每一个** `RUN` **都是启动一个容器**、执行命令、然后提交存储层文件变更。第一层 `RUN cd /app`的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

如果需要改变以后各层的工作目录的位置，那么应该使用 `WORKDIR` 指令。





### ADD指令



`ADD` 指令和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能。

比如 `<源路径>` 可以是一个 `URL`，这种情况下，Docker 引擎会试图去下载这个链接的文件放到 `<目标路径>` 去。下载后的文件权限自动设置为 `600`



**如果 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会*自动*解压缩这个压缩文件到 `<目标路径>` 去。**但在某些情况下，如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用 `ADD` 命令了。



 `COPY` 的语义很明确，就是复制文件而已，而 `ADD` 则包含了更复杂的功能，其行为也不一定很清晰。



## EXPOSE 暴露端口



格式为 `EXPOSE <端口1> [<端口2>...]`。

`EXPOSE` 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用**随机端口映射**时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。



![1546928928274](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1546928928274.png)





### CMD 容器启动命令



之前介绍容器的时候曾经说过，Docker 不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要**指定所运行的程序及参数**。`CMD` 指令就是用于**指定默认的容器主进程的启动命令的**。



在运行时**可以指定新的命令**来替代**镜像设置中的这个默认命令**，比如，`ubuntu` 镜像默认的 `CMD` 是 `/bin/bash`，如果我们直接 `docker run -it ubuntu` 的话，会直接进入 `bash`。我们也可以在运行时指定运行别的命令，如 `docker run -it ubuntu cat /etc/os-release`。这就是用 `cat /etc/os-release` 命令替换了默认的 `/bin/bash` 命令了，输出了系统版本信息。







## Docker容器守护态运行



​	更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。这样可以同时启动多个Docker进程（容器）





### 数据卷Volume



`数据卷` 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

- `数据卷` 可以在容器之间共享和重用
- 对 `数据卷` 的修改会立马生效
- 对 `数据卷` 的更新，不会影响镜像
- `数据卷` 默认会一直存在，即使容器被删除



Docker Container 容器一旦被销毁，容器内的数据将一并删除。客户端上传的图片，文字数据也会一并被删除，容器中的数据不是持久化状态的。

故而**数据卷也被称为 Docker 容器的数据持久化**。



```
-v  v参数是volume数据卷的意思
```



容器中的ROOT目录需要共享存放了Dockfile下的ROOT目录



**-v左边为宿主机的目录，冒号右边是容器中的目录**，这样容器中的文件就挂载在了数据卷中。

```
docker run -p 8080:8080 --name tomcat -d -v /usr/local/docker/tomcat/ROOT:/usr/local/tomcat/webapps/ROOT
```





## 数据库MySQL容器化(持久化)





-e参数支持设置环境变量

-d 守护态运行

```docker
docker pull mysql
(8.0)

docker pull mysql:5.7.22

docker run -p 3306:3306 --name mysql\
-v /usr/local/docker/mysql/conf:/etc/mysql \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7.22



```



**MYSQL 8.0引擎 ， 取消了MyISAM引擎，而保留了支持事务的InnoDB(但查询效率会降低)****



**MYSQL 8.0支持原生分布式数据库**解决方案



**MYSQL 5.7支持NoSQL** 



交互方式运行mysql，进入/etc/mysql/conf.d/mysqldump.cnf

MYSQL默认的最大容量是16M，但利用以上数据卷的方式配置并不能将该配置生效，应多加一步操作，将/etc/mysql/conf.d/mysqldump.cnf的max_allowed_packet=16M这一行放入/etc/mysql/mysql.conf.d



**应预先把所有配置文件放入宿主机中的conf文件夹，因为若默认生成则最大容量不会生效**



**具体步骤：**

1. **在数据卷中先不包含配置文件夹，并交互进入新创立的mysql容器，进入存放最大容量配置的文件，将配置复制，并echo进总配置文件mysqld.cnf最后一行**
2. **将整个mysql配置文件夹复制到宿主机存放配置文件的地方**
3. **关闭当前创建的不持久化的容器，并再用-v 设置配置文件的持久化。**



```
docker run -p 3306:3306 --name mysql\
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7.22

docker exec -it mysql bash

cd /etc/mysql/mysql.conf.d 

ls -al

cat mysqld.cnf

cd ..
ls -al
cat mysqldump.cnf

max____________

cd mysql.conf.d

echo "max____________" mysqld.cnf

cat mysqld.cnf

exit 

docker restart mysql
```



将容器里的附加了最大容量128M的配置文件复制到宿主机



**将容器里的/etc/mysql文件夹全部拷贝到宿主机内的/usr/local/docker/mysql/conf中**，已实现改变了配置的docker容器持久化



```
docker cp mysql:/etc/mysql .
cd mysql 
mv *.* ..
cd ..
rm -fr mysql
```





## 项目的容器化部署



```
docker run -p 8080:8080 --name myshop -v\
/usr/local/docker/tomcat/ROOT/:/usr/local/tomcat/webapps/ROOT\
-d tomcat
```





```
查看容器的日志

docker logs tomcat
```





## Docker Compose



`Compose` 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。从功能上看，跟 `OpenStack` 中的 `Heat` 十分类似。



`Compose` 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」



例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。



`Compose` 恰好满足了这样的需求。它允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。



`Compose` 中有两个重要的概念：

- 服务 (`service`)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。



`Compose` **项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 `Compose` 来进行编排管理。**

**Python非常适合服务端程序**





### 下载

​	

**应用程序往/usr/local放置**

```
curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```



在bin目录下输入ll命令，发现root用户对文件仅有读写权限



-rw-r--r--. 1 root root 11748168 Jan 14 14:32 docker-compose



```
chmod +x docker-compose
docker-compose version 
```





## docker-compose应用



**创建docker-compose.yml**，每个冒号后面都需要有一个空格

```

version: '3'
services:
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat2
    ports:
      - 8080:8080

```





### 启动项目：



```
version: '3'
services:
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat
    ports:
      - 8080:8080
    volumns:
      - /usr/local/docker/myshop/ROOT:/usr/local/tomcat/webapps/ROOT
  mysql:
    restart: always
    image: mysql:5.7.22
    container_name: mysql
    ports:
      - 3306:3306
    environment:
```

