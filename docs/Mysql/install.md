# Docker安装Mysql

## 1、安装Docker

关于Docker安装教程这里不再赘述， 大家自行搜索安装教程；

## 2、下载Mysql镜像

访问 MySQL 镜像库地址：https://hub.docker.com/_/mysql?tab=tags 。

我这里选择的是Mysql5.7的版本，使用下面的命令拉取Docker镜像。

```shell
$ docker pull mysql:latest
```

## 3、查看本地镜像

使用以下命令来查看是否已安装了 Mysql：

```shell
$ docker images
```

![image-20210521115349529](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210521115349529.png)

## 4、运行容器  

我这里将启动命令写入到了shell文件中，启动容器时，直接运行脚本就可以了：

```shell
#!/bin/bash

docker run -d -p 3306:3306 \
--name mysql-5.7 \
-e MYSQL_ROOT_PASSWORD=password \
--privileged=true \
mysql:5.7
```

解释一下上边命令的参数：

- **-p 3306:3306** ：映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过 **宿主机ip:3306** 访问到 MySQL 的服务。
- **MYSQL_ROOT_PASSWORD=password**：设置 MySQL 服务 root 用户的密码。
- **-- name mysql-5.7**：容器的名称
- **mysql:5.7**：代表我们启动mysql5.7的镜像

## 5、进入容器并登录到Mysql

进入容器命令：

```shell
$ docker exec -it mysql bash
```

登录Mysql命令：

```shell
mysql -u root -p
```

### 6、添加远程登录用户

```shell
CREATE USER 'user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';
```

