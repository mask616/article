# Linux下安装JDK

## 一、下载JDK安装包

Oracle官网下载JDK路径：https://www.oracle.com/java/technologies/javase-downloads.html

共享资源下载：http://oss.maskeasy.com/share/jdk-8u281-linux-x64.tar.gz

将压缩包下载至`/server`文件夹下

## 二、安装JDK

### 1.解压JDK安装包

解压安装文件

```shel
tar -zxvf  jdk-8u271-linux-x64.tar.gz
```

### 2.添加配置环境变量

编辑器打开配置文件：

```
vim /etc/profile
```

添加以下环境变量：

在`unset i `之前新增以下内容：

```
export JAVA_HOME=/server/jdk1.8.0_281
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

按 Esc 退出编辑模式，输入 :wq 保存并退出

生效环境变量：

```shell
source /etc/profile
```

### 5.检查安装成功

```shell
java -version
```

## 三、卸载JDK

### 1.检查是否安装了jdk

```shell
java -version
```

### 2.查看JDK安装路径

```
which java
```

### 3.卸载JDK

```
# rm -rf JDK路径
rm -rf /usr/jdk/jdk1.8.0_172/
```

### 4.删除配置环境变量

编辑器打开配置文件：

```
vim /etc/profile删除配置的环境变量：
```

```
export JAVA_HOME=/server/jdk1.8.0_281
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```


按 Esc 退出编辑模式，输入 `:wq`保存并退出

生效环境变量：

```
source /etc/profile
```

