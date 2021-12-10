# Centos7使用别名无密码登录

## 一、serverA 免密登录 serverB 原理

- 首先在 serverA 上生成一对秘钥（ssh-keygen）；

- 将公钥拷贝到 serverB，重命名 authorized_keys
  serverA 向 serverB 发送一个连接请求，信息包括用户名、ip；

- serverB 接到请求，会从 authorized_keys 中查找，是否有相同的用户名、ip，如果有 serverB 会随机生成一个字符串
  然后使用公钥进行加密，再发送个 serverA；

- serverA 接到 serverB 发来的信息后，会使用私钥进行解密，然后将解密后的字符串发送给 serverB；

- serverB 接到 serverA 发来的信息后，会给先前生成的字符串进行比对，如果一直，则允许免密登录；



## 二、Centos7 默认安装了 ssh服务



## 三、serverA 生成秘钥，遇到提示直接敲回车即可



```powershell
CentOS7 默认使用RSA加密算法生成密钥对，保存在~/.ssh目录下的id_rsa（私钥）和id_rsa.pub（公钥）。也可以使用“-t DSA”参数指定为DSA算法，对应文件为id_dsa和id_dsa.pub，
密钥对生成过程会提示输入私钥加密密码，可以直接回车不使用密码保护。
```

```
[root[@localhost ]() ~]# ssh-keygen Generating public/private rsa key pair. 
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:/gGrlDJN5euMS5aai5feBkEI/0WjEnzPzx1xGtdkKG4
root@localhost.localdomain

The key's randomart image is:
+---[RSA 2048]----+
|.o..  o    +o  |
| .o.oo .  + +..  |
|  oo.o. .. B    |
|  o..oo  E    |
|   ...oSo .    |
|   .o +o+.    |
|   ooB + .    |
|  .oX.= . .    |
|  .o=o=.o .    |
+----[SHA256]-----+
[root[@localhost]() ~]# 
```

> 在centos 8.0，生成密钥对的命令不可用，使用`ssh-keygen -t rsa -f ~/.ssh/id_rsa`生成密钥对。

## 四、移动 id_rsa.pub 文件

将 serverA ~/.ssh目录中的 id_rsa.pub 这个文件拷贝到你要登录的 serverB 的~/.ssh目录中

```shell
scp ~/.ssh/id_rsa.pub 192.168.0.101:~/.ssh/
```


然后在 serverB运行以下命令来将公钥导入到~/.ssh/authorized_keys这个文件中

```shell
cat ~/.ssh/id_rsa.pub >>  ~/.ssh/authorized_keys
```

## 五、配置别名登录

修改host文件，在`/etc/hosts`文件中添加一下内容

```powershell
192.168.0.101 G1
```

## 六、测试

```powershell
ssh G1
```

ssh-agent是用于管理密钥，ssh-add用于将密钥加入到ssh-agent中，SSH可以和ssh-agent通信获取密钥，这样就不需要用户手工输入密码了。

顺序执行以上两条命令后就可以用ssh免密登录远程机器了，但这个配置只对当前会话生效，会话关闭或机器重启后都需要重新执行这两条命令。将命令放到~/.bash_profile中，就可以免去每次输入的麻烦。

## 七、简化登录命令

在`/root/.bashrc`文件中加入下面内容，加入以下内容之后需要重启服务器：

```
alias G1='ssh root@G1'
```

加入以上内容之后，我们就可以直接在命令行输入`G1`登录服务器。

## 附：

### 1.如果使用ssh命令登录，提示权限不足， 进行如下操作：

将~/.ssh目录下文件权限设置成600（在serverB上设置）

```
chmod 600 ~/.ssh/*
```

将~/.ssh 文件夹权限设置成700

```
chmod 700 ~/.ssh
```

将家目录权限设置成700

```
chmod 700 $HOME
```

### 2.如果出现如下图所示的异常，执行下边的命令:

```powershell
chmod 600 ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
```

![image-20211019143304532](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20211019143304532.png)

### 3.如果出现如下图的异常， 分别执行下边的代码：

```powershell
eval `ssh-agent`
ssh-add
```

![image-20211019143323430](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20211019143323430.png)

