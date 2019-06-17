## AWS

[TOC]



#### AWS云服务器价格计算器

AWS  WEB 价格计算器网址 <https://calculator.s3.amazonaws.com/index.html>

### 一 创建 EC2(云服务器)

#### 创建步骤一: 进入 控制台，选择系统镜像

![GIFade](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/AWS/EC2_create.gif)

#### 创建步骤二：设置示例类型，配置实例，选择存储，配置安全组，创建实例

![GIF-234](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/AWS/EC2-set.gif)

#### 创建步骤三 ：分配弹性IP，并绑定到主机上。

![image](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/AWS/aws-bind-elastic-ip.gif)

#### 注意事项

> 1. 数据卷选择需要根据需求选择，选择错了会导致费用比较高，具体比较见下文
> 2. 一定要绑定弹性 IP，如果我们没有绑定弹性 IP，默认在实例重启后，公网 IP 是会变化的，如果我们依赖于公网 IP 提供服务的话，这是会很糟糕的，所以我们需要绑定 弹性IP，默认内网IP是不会发生改变的。
> 3. 默认的登录用户 是 `centos` ,是通过秘钥登录的(我们创建的时候指定了秘钥)。

####  EC2  数据卷选择

EBS 各个卷的配置与性能

|                       |                        固态硬盘 (SSD)                        |                       硬盘驱动器 (HDD)                       |                                                              |                                                              |
| :-------------------- | :----------------------------------------------------------: | :----------------------------------------------------------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **卷类型**            |                     通用型 SSD (`gp2`)*                      |                   预配置 IOPS SSD (`io1`)                    | 吞吐优化 HDD (`st1`)                                         | Cold HDD (`sc1`)                                             |
| **描述**              |       平衡价格和性能的通用 SSD 卷，可用于多种工作负载        |  最高性能 SSD 卷，可用于任务关键型低延迟或高吞吐量工作负载   | 为频繁访问的吞吐量密集型工作负载设计的低成本 HDD 卷          | 为不常访问的工作负载设计的最低成本 HDD 卷                    |
| **使用案例**          | 建议用于大多数工作负载系统启动卷虚拟桌面低延迟交互式应用程序开发和测试环境 | 需要持续 IOPS 性能或每卷高于 16,000 IOPS 或 250 MiB/s 吞吐量的关键业务应用程序大型数据库工作负载，如：MongoDBCassandraMicrosoft SQL ServerMySQLPostgreSQLOracle | 以低成本流式处理需要一致、快速的吞吐量的工作负载大数据数据仓库日志处理不能是启动卷 | 适合大量不常访问的数据、面向吞吐量的存储最低存储成本至关重要的情形不能是启动卷 |
| **API 名称**          |                            `gp2`                             |                            `io1`                             | `st1`                                                        | `sc1`                                                        |
| **卷大小**            |                        1 GiB - 16 TiB                        |                        4 GiB - 16 TiB                        | 500 GiB - 16 TiB                                             | 500 GiB - 16 TiB                                             |
| **最大IOPS\**/卷**    |                          16,000***                           |                          64,000****                          | 500                                                          | 250                                                          |
| **最大吞吐量/卷**     |                         250 MiB/s***                         |                         1,000 MiB/s†                         | 500 MiB/s                                                    | 250 MiB/s                                                    |
| **最大IOPS/实例**††   |                            80,000                            |                            80,000                            | 80,000                                                       | 80,000                                                       |
| **最大吞吐量/实例**†† |                         1,750 MiB/s                          |                         1,750 MiB/s                          | 1,750 MiB/s                                                  | 1,750 MiB/s                                                  |
| **管理性能属性**      |                             IOPS                             |                             IOPS                             | MiB/s                                                        | MiB/s                                                        |

**新加坡可用区**价格表(时间：2019-05-30)

| 数据卷类型           | 容量        | IOPS | 最大吞吐量       |             |
| -------------------- | ----------- | ---- | ---------------- | ----------- |
| gp2                  | 50GB        | 150  | 128 MBs/sec      | $6/month    |
| IOPS SSD (`io1`)     | 50GB        | 150  | 37.5 MBs/sec     | $17.7/month |
| 吞吐优化 HDD (`st1`) | 500GB(最小) | —    | 19.53125 MBs/sec | $27/month   |

根据上面的对比，建议选择`gp2` (性价比高)。`AWS` 云服务器的硬盘的 `IOPS` 是跟硬盘有关的。但是 EBS 卷支持生产期间的实时配置更改。您可以在不中断服务的情况下修改卷类型、卷大小和 IOPS 容量。EC2 默认创建的卷的类型是 `gp2`。

####  EC2 创建快照计划任务

在实际生产环境中，定时创建一个快照也是一个对数据备份的好办法，所以接下来我们来讲讲创建快照计划任务。

方法一： 生命周期管理器  官方文档：<https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/snapshot-lifecycle.html>

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/AWS/EC2-create-snapshot.gif)



方法二： 通过`cloudwatch` 创建 规则(rules)

通过这种方式创建的快照，是不会自动删除的，就是会将创建的快照一直保存。

![1559206665634](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/AWS/1559206665634.png)

#### 设置 root 用户密码

AWS EC2 默认创建的用户是 `centos`,是使用秘钥进行登录的，我们考虑到我们需要`root`用户,虽然我们不常用到，但是为了以备后期使用，是服务器的安全，我们这里会配置 `root` 用户密码，但不允许 root 用户远程登录。

具体操作

```bash
[centos@ip-172-31-21-255 ~]$ sudo passwd  root
Changing password for user root.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
[centos@ip-172-31-21-255 ~]$ su root 
Password: 
[root@ip-172-31-21-255 centos]# id
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@ip-172-31-21-255 centos]# 
```

如果我们需要通过 root 来进行远程登陆的话，可以选择远程登陆使用 密码登陆

```bash
# 将 sshd配置文件/etc/ssh/sshd_config 参数 PasswordAuthentication no 更改为 yes
sed -i '/^PasswordAuthentication/s/no/yes/' /etc/ssh/sshd_config
systemctl  restart  sshd
# 更改之后，我们只能通过密码进行登陆了，而不能通过密钥登陆了。这时我们需要给 centos 设置密码，然后通过密码登陆
```

如果我们想要通过 root 通过密钥进行远程登陆的话，我们需要先在 xshell 生成密钥，并将公钥存放在 root 用户的 `/root/.ssh/authorized_keys` 里，然后进行登陆。

生成密钥，并获取公钥(要记住设置的密码，后面要使用)

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/AWS/ssh-pub-login.gif)





### 二、AWS 注意事项

#### 弹性IP限制

注意在每个地域，每个账户的默认可以申请的弹性IP是 5个。如果我们的计划需要使用的弹性IP超过了5个，我们需要提前进行申请，将限制提高。

#### 付费类型

一种是RI，另外一种是OnDemand

RI预付费

OnDemand后付费

#### EC2实例有两种根设备类型

1、实例存储（本地），系统和磁盘再同一主机上

​     Instances storage，当删除了实例，数据卷也释放了

2、EBS存储（网盘），EBS可能与云主机不在一台物理机上，可能在其他物理机上。

​      Elastic Block Storage，当删除了实例，EBS 卷可以单独保存。

#### EC2 实例数量限制

EC2实例在申请超过20台后，会有数量限制。

#### AWS 使用RDS注意时区参数

使用 AWS 的 RDS ，记住需要创建一个参数组，默认的参数组是不可以修改的，我们新建的参数组是可以修改的。 

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/AWS/aws-rds-time-zone.png)

#### AWS ELB

ELB 是不可以设置黑名单的，也是不可以开启 gzip 的。



有关AWS 的一些其他文章可查看 [博客园-宋某人](https://www.cnblogs.com/syaving/)

