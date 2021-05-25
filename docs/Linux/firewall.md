# 如何在CentOS7系统中使用iptables

[toc]

## 概述

CentOS7系统默认的防火墙是Filewalld。但是，仍有大量用户习惯于在CentOS7系统中使用iptables。本文以CentOS7.4为例，说明在CentOS7系统中如何安装并使用iptables。

 

## 详细信息

本文主要介绍如下几点内容。

- 禁止Filewalld开机启动
- 安装iptables
- 启动iptables并设置为开机启动
- 查看并修改iptables默认规则

 

### 禁止Filewalld开机启动

为了防止与iptables冲突，您必须先禁止Filewalld开机启动。

1. 连接到服务器。

2. 执行如下命令，查看服务状态。

   ```
   systemctl status firewalld
   ```

   系统显示类似如下，active字段表示服务处于运行状态，inactive字段表示服务处于关闭状态。

   ![image-20210525201029388](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525201029388.png)

3. 当服务处于active状态，运行以下命令关闭Firewalld服务。

   ```
   systemctl stop firewalld
   ```

4. 执行如下命令，禁止Filewalld开机启动。

   ```
   systemctl disable firewalld
   ```

 

### 安装iptables

执行如下命令，安装iptables。

```
yum install -y iptables-services
```

 

### 启动iptables并设置为开机启动

1. 执行如下命令，启动iptables。

   ```
   systemctl start iptables
   ```

2. 执行如下命令，查看iptables是否成功启动。

   ```
   systemctl status iptables
   ```

   系统显示类似如下，说明iptables已经成功启动。

   ![image-20210525201048324](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525201048324.png)

3. 执行如下命令，设置iptables开机启动。

   ```
   systemctl enable iptables.service
   ```

4. 设置完成后，执行如下命令，重启实例验证配置。

   ```
   systemctl reboot
   ```

 

### 查看并修改iptables默认规则

执行`iptables -L`命令，查看iptables默认规则，发现在默认规则下，INTPUT链允许来自任何主机的访问。可以参考如下步骤修改默认规则。

1. 如果之前已经设置过规则，建议执行如下命令，备份原有的iptables文件，避免之前设置的规则丢失。

   ```
   cp -a /etc/sysconfig/iptables /etc/sysconfig/iptables.bak
   ```

2. 执行如下命令，清空所有规则。

   ```
   iptables -F
   ```

3. 根据业务需求添加规则，放行或者禁用端口。示例：依次执行如下命令，放行80端口和22端口。

   ```
   iptables -I INPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT
   iptables -I INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT
   ```

   示例：依次执行如下命令，添加规则，使INPUT链拒绝所有请求，即ECS实例会拒绝所有请求。如果是线上业务请勿直接操作，会直接中断业务。

   ```
   iptables -P INPUT DROP
   ```

4. 执行如下命令，确认新规则生效。

   ```
   iptables -L
   ```

5. 执行如下命令，保存添加的规则。

   ```
   iptables-save > /etc/sysconfig/iptables
   ```

