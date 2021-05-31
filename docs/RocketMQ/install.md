# 使用Docker安装RocketMQ

## 1、下载RockerMQ-Docker项目

下载RockerMQ-Docker项目，该项目里边包含了我们构建以及启动项目的脚本，甚至连路径挂载都已经配置好了。

使用git的下载方式：

```shell
git clone https://github.com/apache/rocketmq-docker.git
```

## 2、构建Docker镜像

我们先切换到image-build目录，然后运行build脚本。

```shell
cd image-build
sh build-image.sh 4.7.1 centos
```
这时我们使用`docker images`命令就可以看到我们build好的镜像：

![image-20210531142915974](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210531142915974.png)

## 3、生成启动脚本

使用stage.sh脚本文件生成启动脚本。执行stage.sh脚本文件后， 在当前目录使用`cd ./stage/4.7.1/templates`切换到启动脚本所在的目录，就可以看到生成的脚本文件。

```shell
sh stage.sh 4.7.1
```

## 4、修改配置文件

这里我们自定义一下broker的配置文件，broker配置如下：

```properties
brokerClusterName=DefaultCluster
brokerName=broker-a
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=ASYNC_MASTER
flushDiskType=SYNC_FLUSH
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
```

重要参数解释：

> autoCreateTopicEnable : 是否开启自动创建topic
> autoCreateSubscriptionGroup：是否自动创建订阅组

## 5、修改启动脚本

这里主要是修改将Docker RocketMQ的日志文件，数据文件映射到本地，以及自定义配置文件。

```shell
docker run -d -v /server/docker/images/rocketmq/logs:/root/logs -v /server/docker/images/rocketmq/data:/root/store -v /server/docker/images/rocketmq/conf/broker.conf:/home/rocketmq/rocketmq-4.7.1/conf/broker.conf --name rmqbroker --link rmqnamesrv:namesrv -e "NAMESRV_ADDR=namesrv:9876" -p 10909:10909 -p 10911:10911 -p 10912:10912 apacherocketmq/rocketmq:4.7.1${TAG_SUFFIX} sh mqbroker -c /home/rocketmq/rocketmq-4.7.1/conf/broker.conf
```

## 6、启动RocketMQ

```she
cd ./stage/4.7.1/templates
./play-docker.sh centos
```

## 7、TODO 修改RocketMQ启动内存

```java
// todo
```
