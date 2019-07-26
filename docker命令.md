docker命令





3、安装

```
//安装最新版本
$ sudo yum install docker-ce


//或者安装指定版本
$ yum list docker-ce --showduplicates | sort -r

    //显示有以下版本
    docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
    
//指定一个版本进行安装
$ sudo yum install docker-ce-<VERSION STRING>
```





4、启动doker

```
$ sudo systemctl start docker
```

5、通过运行hello-world镜像验证安装是否成功

```
$ sudo docker run hello-world
```





scp -r fastdfs root@172.16.2.21: