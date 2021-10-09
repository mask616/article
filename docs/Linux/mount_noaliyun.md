## 二、在服务器挂载云盘

     一块全新的数据盘挂载到ECS实例后，您必须创建并挂载至少一个文件系统。本示例使用I/O优化实例，操作系统为Alibaba Cloud Linux 2.1903 LTS 64位，为一块新的20 GiB数据盘（设备名为/dev/vdb）创建一个MBR格式的单分区，挂载的是ext4文件系统。

### 前提条件

    随实例一起购买的数据盘，已自动挂载到该实例。单独购买的数据盘必须挂载到实例后才能格式化，详情请参见[挂载数据盘](https://help.aliyun.com/document_detail/25446.htm?spm=a2c4g.11186623.2.7.36ef1405wmw7TX#concept-llz-b4c-ydb)。

>  **说明** 此处挂载操作指在控制台将云盘挂载到ECS实例，并非在ECS实例操作系统内通过mount命令挂载文件系统。

#### 背景信息

本文操作仅适用小于等于2 TiB的数据盘。大于2TiB的数据盘分区必须使用GPT格式，请参见[分区格式化大于2 TiB数据盘](https://help.aliyun.com/document_detail/34377.htm?spm=a2c4g.11186623.2.8.36ef1405wmw7TX#concept-i15-qpc-ydb)。

数据盘的设备名默认由系统分配，命名规则如下所示：

- I/O优化实例的数据盘设备名从/dev/vdb递增排列，包括/dev/vdb~/dev/vdz。
- 非I/O优化实例的数据盘设备名从/dev/xvdb递增排列，包括/dev/xvdb~/dev/xvdz。

格式化操作可能存在如下风险：

- 磁盘分区和格式化是高风险行为，请慎重操作。本文操作仅适用处理一块全新的数据盘，如果您的数据盘上有数据，请务必为数据盘创建快照，避免数据丢失。详情请参见[创建一个云盘快照](https://help.aliyun.com/document_detail/25455.htm?spm=a2c4g.11186623.2.9.36ef1405wmw7TX#concept-eps-gbl-xdb)。
- 云服务器ECS仅支持**数据盘**分区操作，不支持**系统盘**分区操作。如果您强行使用第三方工具对系统盘做分区操作，可能引发系统崩溃和数据丢失等未知风险。仅允许在扩容系统盘后做扩展分区或新增分区操作，具体操作请参见[在线扩容云盘（Linux系统）](https://help.aliyun.com/document_detail/113316.htm?spm=a2c4g.11186623.2.10.36ef1405wmw7TX#concept-syg-jxz-2hb)。

>  **说明** 本示例操作命令同样适用于CentOS 7系统。

### 步骤一：为数据盘创建MBR分区

1. 远程连接ECS实例。

   如何连接ECS实例，具体操作请参见[通过密码或密钥认证登录Linux实例](https://help.aliyun.com/document_detail/147650.htm?spm=a2c4g.11186623.2.11.36ef1405wmw7TX#task-2367667)。

2. 查看实例上的数据盘信息。

   运行以下命令：

   ```
   fdisk -l
   ```

   运行结果如下所示。![image-20210525195407772](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525195407772.png)

   > **说明** 执行命令后，如果不存在/dev/vd*（/dev/vd*指您新购买的那块数据盘，本示例中为/dev/vdb），请确认数据盘是否已经挂载到实例。关于如何在控制台挂载数据盘，请参见[挂载数据盘](https://help.aliyun.com/document_detail/25446.htm?spm=a2c4g.11186623.2.13.36ef1405wmw7TX#concept-llz-b4c-ydb)。

3. 依次运行以下命令，创建一个分区。

   1. 运行以下命令分区数据盘。

      ```
      fdisk -u /dev/vdb
      ```

   2. 输入`p`查看数据盘的分区情况。

      本示例中，数据盘没有分区。

   3. 输入`n`创建一个新分区。

   4. 输入`p`选择分区类型为主分区。

      **说明** 创建一个单分区数据盘可以只创建主分区。如果要创建四个以上分区，您应该至少选择一次e（extended），创建至少一个扩展分区。

   5. 输入分区编号，按回车键。

      本示例中，仅创建一个分区，直接按回车键，采用默认值1。

   6. 输入第一个可用的扇区编号，按回车键。

      本示例中，直接按回车键，采用默认值2048。

   7. 输入最后一个扇区编号，按回车键。

      本示例中，仅创建一个分区，直接按回车键，采用默认值。

   8. 输入`p`查看该数据盘的规划分区情况。

   9. 输入`w`开始分区，并在完成分区后退出。

   运行结果如下所示。

   ![image-20210525195440378](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525195440378.png)

4. 查看新分区信息。

   运行以下命令：

   ```
   fdisk -lu /dev/vdb
   ```

   运行结果如下所示，如果出现/dev/vdb1的相关信息，表示新分区已创建完成。

   ![image-20210525195551810](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525195551810.png)

### 步骤二：为分区创建文件系统

在新分区上创建一个文件系统。根据您的需求运行以下任一命令，创建文件系统。

- 创建一个ext4文件系统，运行以下命令。

  ```
  mkfs -t ext4 /dev/vdb1
  ```

- 创建一个xfs文件系统，运行以下命令。

  ```
  mkfs -t xfs /dev/vdb1
  ```

本示例中，创建一个ext4文件系统。

![image-20210525195514937](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525195551810.png)

### 步骤三：配置/etc/fstab文件并挂载分区

在/etc/fstab中写入新分区信息，启动开机自动挂载分区。

>  **注意** 由于释放云盘等操作可能会导致其他云盘的设备名变动，建议您在/etc/fstab中使用全局唯一标识符UUID来引用新分区。

1. 备份etc/fstab文件。

   运行以下命令：

   ```
   cp /etc/fstab /etc/fstab.bak
   ```

2. 在/etc/fstab里写入新分区信息。

   - root用户可以运行以下命令直接修改/etc/fstab文件。

     ```
     echo `blkid /dev/vdb1 | awk '{print $2}' | sed 's/\"//g'` /server ext4 defaults 0 0 >> /etc/fstab
     ```

     **说明**

     - Ubuntu 12.04系统不支持barrier，您需要运行`echo '`blkid /dev/vdb1 | awk '{print $3}' | sed 's/\"//g'` /mnt ext4 barrier=0 0 0' >> /etc/fstab`命令。
     - 如果要把数据盘单独挂载到某个文件夹，例如单独用来存放网页，则将命令中`/mnt`替换成所需的挂载点路径。

   - 普通用户可以手动修改/etc/fstab文件。

     1. 运行以下命令查看新分区的UUID。

        ```
        sudo blkid /dev/vdb1
        ```

        运行结果如下所示。

        ```
        /dev/vdb1: UUID="05779a4e-f04f-4eca-97ac-57fd1fda****" TYPE="ext4"
        ```

     2. 运行以下命令编辑/etc/fstab文件。

        ```
        sudo vi /etc/fstab
        ```

     3. 输入`i`进入编辑模式。

     4. 在/etc/fstab文件中写入新分区信息，UUID值请修改为前面步骤中的查询结果。

        ```
        UUID=05779a4e-f04f-4eca-97ac-57fd1fda**** /mnt ext4 defaults 0 0
        ```

     5. 按Esc键，输入`:wq`，按回车键保存并退出。

3. 查看/etc/fstab中的新分区信息。

   运行以下命令：

   ```
   cat /etc/fstab
   ```

   运行结果如下所示。

   ![image-20210525195758798](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525195758798.png)

4. 挂载分区。

   运行以下命令：

   ```
   mount /dev/vdb1 /mnt
   ```

5. 检查挂载结果。

   运行以下命令：

   ```
   df -h
   ```

   运行结果如下所示，如果出现新建文件系统的信息，表示文件系统挂载成功。

   ![image-20210525195921291](https://gitee.com/mask616/images-bed/raw/master/typora-images/image-20210525195921291.png)