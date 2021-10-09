[toc]

# Linux挂载2T以上的磁盘

本文将说明如何在Linux实例上使用Parted工具和e2fsprogs工具分区并格式化一个大容量数据盘。假设需要处理的数据盘是一个新建的3 TiB的空盘，设备名为/dev/vdb。

## 前提

**前提条件：请确认您的Linux实例上已经安装了Parted工具和e2fsprogs工具。**

- 运行以下命令安装Parted工具：

  ```shell
  yum install -y parted
  ```

- 运行以下命令安装e2fsprogs工具：

  ```shell
  yum install -y e2fsprogs
  ```

## 开始挂载

### 1.连接Linux服务器。

略

### 2.查看是否存在数据盘。

运行以下命令：

```shell
fdisk -l
```

运行结果如下所示，应包含数据盘信息。如果没有，表示您未挂载数据盘。

```
Disk /dev/vdb: 3221.2 GB, 3221225472000 bytes, 6291456000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

### 3.使用Parted工具为数据盘进行分区。

#### 3.1 开始分区

```shell
parted /dev/vdb
```

#### 3.2 将默认的MBR分区格式转为GPT分区格式

```shell
mklabel gpt
```

#### 3.3 划分一个主分区，并设置分区的开始位置和结束位置

```shell
mkpart primary 1 100%
```

#### 3.4 检查分区是否对齐

```shell
align-check optimal 1
```

运行结果如下所示：

```
1 aligned
```

>**说明** 如果返回的是1 not aligned，说明分区未对齐，建议您运行以下命令 ，再根据`（<optimal_io_size>+<alignment_offset>）/<physical_block_size>`的公式计算出最佳分区模式的起始扇区值。假设1024为计算得出的推荐扇区值，则您可以运行`mkpart primary 1024s 100%`重新划分一个主分区。
>
>```
>cat /sys/block/vdb/queue/minimum_io_size
>cat /sys/block/vdb/alignment_offset
>cat /sys/block/vdb/queue/physical_block_size

4.运行以下命令，查看分区表。

```
print
```

5.运行以下命令，退出Parted工具。

```
quit
```

Parted工具分区结果如下所示。

![image-20211009163649255](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20211009163649255.png)

### 4.运行以下命令，使系统重读分区表。

```
partprobe
```

### 5.运行以下命令，为/dev/vdb1分区创建一个文件系统。

根据您的需求运行以下任一命令，创建文件系统。

- 创建一个ext4文件系统。

  ```
  mkfs -t ext4 /dev/vdb1
  ```

- 创建一个xfs文件系统。

  ```
  mkfs -t xfs /dev/vdb1
  ```

> **说明**
>
> - 如果数据盘的容量为16 TiB，您需要使用指定版本的e2fsprogs工具包格式化，请参见[附录一：升级e2fsprogs工具包](#附录一：Linux实例升级e2fsprogs工具包)。
> - 如果您要关闭ext4文件系统的lazy init功能，避免该功能对数据盘I/O性能的影响，请参见[附录二：关闭lazy init功能](#附录二：Linux实例关闭lazy init功能)。

### 6.运行以下命令，创建一个名为/test的挂载点。

```
mkdir /test
```

### 7.运行以下命令，将分区/dev/vdb1挂载到/test。

```
mount /dev/vdb1 /test
```

### 8.运行以下命令，查看目前磁盘空间和使用情况。

```
df -h
```

如果返回结果里出现新建文件系统的信息，说明挂载成功，您可以使用新的文件系统了。

![image-20211009163713389](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20211009163713389.png)

### 9.（推荐）在/etc/fstab文件中写入新分区信息，启动开机自动挂载分区。

1. 运行以下命令，备份etc/fstab。

   ```
   cp /etc/fstab /etc/fstab.bak
   ```

2. 运行以下命令，向/etc/fstab里写入新分区信息。

   ```
   echo `blkid /dev/vdb1 | awk '{print $2}' | sed 's/\"//g'` /test ext4 defaults 0 0 >> /etc/fstab
   ```

   **说明**

   - 需要使用root用户运行此命令。如果您使用的是普通用户，可运行`su -`命令切换到root用户，然后运行此命令；或者直接运行`sudo vi /etc/fstab`命令编辑/etc/fstab文件。
   - 建议在/etc/fstab中使用全局唯一标识符UUID来引用新分区。您可以使用**blkid**命令获得新分区的UUID。

3. 运行以下命令，查看/etc/fstab的信息。

   ```
   cat /etc/fstab
   ```

   如果返回结果里出现了写入的新分区信息，说明写入成功。

至此，您已经成功分区并格式化了一个3 TiB数据盘。



## 附录一：Linux实例升级e2fsprogs工具包

如果数据盘容量为16 TiB，您需要使用1.42及以上版本的e2fsprogs工具包完成ext4文件系统格式化。如果e2fsprogs版本低于1.42，会出现如下错误信息。

```
mkfs.ext4: Size of device /dev/vdb too big to be expressed in 32 bits using a blocksize of 4096.            
```

您需要按以下方式安装高版本的e2fsprogs，如本示例中使用的1.42.8。

1. 运行以下命令检查e2fsprogs当前的版本。

   ```
   rpm -qa | grep e2fsprogs
   ```

   运行结果如下所示。

   ![image-20211009163738447](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20211009163738447.png)

   如果当前版本低于1.42，按以下步骤安装软件。

2. 运行以下命令下载1.42.8版本的e2fsprogs。您可以在 [e2fsprogs](https://www.kernel.org/pub/linux/kernel/people/tytso/e2fsprogs/v1.42.8/?spm=a2c4g.11186623.2.14.Pb5baW)找到最新的软件包。

   ```
   wget https://www.kernel.org/pub/linux/kernel/people/tytso/e2fsprogs/v1.42.8/e2fsprogs-1.42.8.tar.gz
   ```

3. 依次运行以下命令，编译高版本的工具。

   3.1 运行以下命令，解压软件包。

   ```
   tar xvzf e2fsprogs-1.42.8.tar.gz
   ```

   3.2 运行以下命令，进入软件包目录。

   ```
   cd e2fsprogs-1.42.8
   ```

   3.3 运行以下命令，生成Makefile文件。

   ```
   ./configure
   ```

   3.4 运行以下命令，编译e2fsprogs。

   ```
   make
   ```

   3.5 运行以下命令，安装e2fsprogs。

   ```
   make install
   ```

4. 运行以下命令检查是否成功更新版本。

   ```
   rpm -qa | grep e2fsprogs
   ```

## 附录二：Linux实例关闭lazy init功能

ext4文件系统默认开启lazy init功能。该功能开启时，实例会发起一个线程持续地初始化ext4文件系统的metadata，从而延迟metadata初始化。所以在格式化数据盘后的近期时间内，云盘的IOPS性能会受到影响，IOPS性能测试的数据会明显偏低。

如果您需要在格式化以后马上测试数据盘性能，请运行以下命令在格式化文件系统时关闭lazy_init功能。

```shell
mke2fs -O 64bit,has_journal,extents,huge_file,flex_bg,uninit_bg,dir_nlink,extra_isize -E lazy_itable_init=0,lazy_journal_init=0   /dev/vdb1
```

> **说明** 关闭lazy init功能后，格式化的时间会大幅度地延长，格式化32 TiB的数据盘可能需要10分钟~30分钟。请您根据自身的需要选择是否使用lazy init功能。