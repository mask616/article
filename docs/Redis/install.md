# 使用Docker安装Redis

[toc]



## 1、找到合适的Redis Docker镜像

- Docker hub地址 ：https://hub.docker.com/_/redis?tab=tags&page=1&ordering=last_updated

通过官网得知，目前的稳定版本是5.0版本， 所以这里作者选择安装5.0版本的镜像。

![image-20210525151804580](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525151804580.png)

## 2、使用docker安装Redis

使用如下的命令下载Dokcer镜像:

```shell
$ sudo docker pull redis:5.0
```

下载好以后使用docker images查看镜像是否已经下载好，如下图所示我们已经成功下载了镜像:

![image-20210525152511754](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525152511754.png)

## 3、准备Redis的配置文件

在https://download.redis.io/releases/这个页面找到对应版本压缩文件， 将文件加压后会看到redis配置文件：redis.cong

![image-20210525154801056](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525154801056.png)

## 4、配置redis.conf配置文件

修改redis.conf配置文件：
主要配置的如下：

> bind 127.0.0.1 #注释掉这部分，使redis可以外部访问<br>
> daemonize no#用守护线程的方式启动<br>
> requirepass 你的密码#给redis设置密码<br>
> appendonly yes#redis持久化　　默认是no<br>
> tcp-keepalive 300 #防止出现远程主机强迫关闭了一个现有的连接的错误 默认是300

**一定要仔细检查上边的配置是否配置正确了， 笔者这里就是bind这个地方没有配置对， 搞了好久。**

## 5、创建本地与docker映射的目录，即本地存放的位置

在这里我创建可两个文件夹分别是config文件夹和data文件夹。 其中config文件夹，用于存放redis.conf配置文件；data文件夹用于存放redis的数据。

![img](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525193522820.png)

## 6、启动Docker Redis

**请读者注意， -v参数要修改成你自己存放配置文件和Redis数据的路径。Docker容器内的路径可以不用修改。**

```shell
 sudo docker run -p 6379:6379 \
 --name redis-5.0 \
 --privileged=true \
 -v /server/docker/images/redis/config/redis.conf:/etc/redis/redis.conf  \
 -v /server/docker/images/redis/data:/data \
 -d redis:5.0 redis-server /etc/redis/redis.conf \
 --appendonly yes 
```

参数解释：

> -p 6379:6379 : 把容器内的6379端口映射到宿主机6379端口<br>
> -v /data/redis/redis.conf:/etc/redis/redis.conf：把宿主机配置好的redis.conf放到容器内的这个位置中<br>
> -v /data/redis/data:/data：把redis持久化的数据在宿主机内显示，做数据备份<br>
> redis-server /etc/redis/redis.conf：这个是关键配置，让redis不是无配置启动，而是按照这个redis.conf的配置启动<br>
> –appendonly yes：redis启动后数据持久化<br>
> -privileged：使用该参数，container内的root拥有真正的root权限。否则，container内的root只是外部的一个普通用户权限。<br>

## 7、验证

我们使用redis-cli进行验证， 验证结果如下：

![image-20210525192717956](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525192717956.png)

通过截图我们可以看出Redis已经安装成功了。