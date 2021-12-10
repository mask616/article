# 修改阿里云服务器实例名

阿里云服务器修改主机名即ID(登陆时root@后显示名)

## 一、登陆服务器

用远程登陆工具登陆服务器

## 二、修改实例名

在任意目录执行以下命令： 

```shell
hostnamectl set-hostname 要修改的名字
```



![image-20211019144241132](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20211019144241132.png)

执行完上面的命令之后需要重启服务器。

## 三、再次登陆远程服务器查看实例名

![image-20211019144414112](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20211019144414112.png)