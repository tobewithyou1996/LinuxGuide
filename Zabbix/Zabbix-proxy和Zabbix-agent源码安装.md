## 一 、Zabbix Proxy

#### 概述

Zabbix proxy 是一个可以从一个或多个受监控设备采集监控数据并将信息发送到 Zabbix server 的进程，主要是代表 Zabbix server 工作。 所有收集的数据都在本地缓存，然后传输到 proxy 所属的 Zabbix server。

部署Zabbix proxy 是可选的，但可能非常有利于分担单个 Zabbix server 的负载。 如果只有代理采集数据，则 Zabbix server 上会减少 CPU 和磁盘 I/O 的开销。

Zabbix proxy 是无需本地管理员即可集中监控远程位置、分支机构和网络的理想解决方案。

Zabbix proxy 需要使用独立的数据库。

###  Zabbix proxy安装

####  下载编译

下载

```bash
cd  /tmp && wget   https://jaist.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/4.0.9/zabbix-4.0.9.tar.gz
```

解压

```
tar  -xzf  zabbix-4.0.9.tar.gz 
```

创建用户和组并创建安装目录

```bash
sudo groupadd zabbix
sudo useradd -g zabbix zabbix
sudo mkdir  /opt/zabbix-proxy
sudo chown  zabbix:zabbix /opt/zabbix-proxy
```

安装依赖包

```bash
yum install -y  mysql-devel net-snmp net-snmp-devel  libssh2-devel 
```

编译安装

```bash
cd  zabbix-4.0.9
# 如果想使用其它参数和数据库，使用 ./configure --help 查看选项和参数,使用mysql 作为 proxy 的数据库
./configure --prefix=/opt/zabbix-proxy --enable-proxy  --with-net-snmp --with-mysql --with-ssh2
make install 
```

####  创建Zabbix  proxy数据库并导入数据

Zabbix  proxy 是将数据储存在本地，然后传输到 Zabbix Server 的。所以我们需要创建 Zabbix proxy 的数据库。

```
# 创建数据库
create database zabbix_proxy character set utf8 collate utf8_bin;
# 创建用户
grant all privileges on zabbix_proxy.* to zabbix_fy@localhost identified by 'password';
```

导入数据，zabbix proxy 不需要将所有的数据库数据都导入，只需要导入 `schema.sql`

```
mysql -u zabbix_fy  -p --database zabbix_proxy </tmp/zabbix-4.0.9/database/mysql/schema.sql
```

####  更改Zabbix proxy 配置文件

默认配置文件是 位于  安装目录的`./etc/zabbix_proxy.conf`。

默认启用的是主动模式，默认监听端口： 10051。参数详解：[官方文档](https://www.zabbix.com/documentation/4.0/zh/manual/appendix/config/zabbix_proxy)



```
Server=# 填入zabbix server 的ip
ServerPort= # zabbix server 监听的端口，默认为 10051
Hostname=#zabbix Proxy 的名称
DBHost= # 数据库地址
DBName=zabbix_proxy # 数据库名称
DBUser=zabbix_fy # 用户名
DBPassword=sRW123456 # 密码
ProxyOfflineBuffer=24 # 如果连接不到zabbix-server，数据保存多久。
```





#### 设置为 `systemd` 服务

创建 `/usr/lib/systemd/system/zabbix-proxy.service` 文件。并添加以下内容：

```
[Unit]
Description=Zabbix Proxy
After=syslog.target
After=network.target

[Service]
User=zabbix
Group=zabbix
Environment="CONFFILE=/opt/zabbix-proxy/etc/zabbix_proxy.conf"
Type=forking
Restart=on-failure
PIDFile=/tmp/zabbix_proxy.pid
KillMode=control-group
ExecStart=/opt/zabbix-proxy/sbin/zabbix_proxy -c $CONFFILE
ExecStop=/bin/kill -SIGTERM $MAINPID
RestartSec=10s
TimeoutSec=0

[Install]
WantedBy=multi-user.target
```

启动服务并设置为开机自启

```bash
# sudo  systemctl  restart  zabbix-proxy
# sudo  systemctl  enable  zabbix-proxy
```

开放对应的端口

```bash
sudo firewall-cmd --add-port=10051/tcp  --permanent 
sudo firewall-cmd --reload 
```



### Zabbix Proxy 安装报错与解决办法

这里报的错都是由于依赖包没有安装，导致编译时报错。

**错误一**

```
checking for the linux kernel version... unknown family (3.10.0-862.14.4.el7.x86_64)
checking size of void *... 8
checking for mysql_config... no
checking for mariadb_config... no
configure: error: MySQL library not found
```

解决办法

```bash
yum install -y  mysql-devel
```

**错误二**

```bash
checking for Zabbix server/proxy database selection... ok
checking for multirow insert statements... yes
checking for pkg-config... /usr/bin/pkg-config
checking pkg-config is at least version 0.9.0... yes
checking for net-snmp-config... no
configure: error: Invalid Net-SNMP directory - unable to find net-snmp-config
```

解决办法

```bash
yum  install  net-snmp net-snmp-devel  -y
```

**错误三**

```bash
checking for main in -lnetsnmp... yes
checking for localname in struct snmp_session... yes
checking for SSH2 support... no
configure: error: SSH2 library not found
```

解决办法

```bash
 yum install libssh2-devel -y
```

##  二、Zabbix agent

### Zabbix agent安装

#### 下载编译

下载

```bash
cd  /tmp && wget   https://jaist.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/4.0.9/zabbix-4.0.9.tar.gz
```

解压

```
tar  -xzf  zabbix-4.0.9.tar.gz 
```

创建用户和组并创建安装目录

```bash
sudo groupadd zabbix
sudo useradd -g zabbix zabbix
sudo mkdir  /opt/zabbix-agent
sudo chown  zabbix:zabbix /opt/zabbix-agent
```

编译安装

```bash
cd  zabbix-4.0.9
./configure  --prefix=/opt/zabbix-agent --enable-agent 
```



#### 更改Zabbix agent 配置文件

默认配置文件是 位于  安装目录的`./etc/zabbix_agentd.conf`。

我们一般需要更改以下参数：

```bash
Server: 设置该值为 Zabbix Server IP.默认为 127.0.0.1
ServerActive：设置该值为 Zabbix Server IP，如果 Zabbix Server 不是使用的默认10051端口，我们可以在此加上端口号，形如：192.168.12.234:11051,默认为127.0.0.1
Hostname ：设置为主机的主机名,默认为 zabbix server 
LogFileSize=1   日志文件超过 1M 就进行切割。值为 0时表示不切割日志。 默认为 1
EnableRemoteCommands：是否开启远程命令 默认为 0
```



#### 设置为 `systemd` 服务

创建 `/usr/lib/systemd/system/zabbix-agent.service` 文件。并添加以下内容：

```
[Unit]
Description=Zabbix Agent
After=syslog.target
After=network.target

[Service]
User=zabbix
Group=zabbix
Environment="CONFFILE=/opt/zabbix-agent/etc/zabbix_agentd.conf"
Type=forking
Restart=on-failure
PIDFile=/tmp/zabbix_agentd.pid
KillMode=control-group
ExecStart=/opt/zabbix-agent/sbin/zabbix_agentd -c $CONFFILE
ExecStop=/bin/kill -SIGTERM $MAINPID
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

启动服务并设置为开机自启

```bash
sudo  systemctl  restart  zabbix-agent
sudo  systemctl  enable  zabbix-agent
```

开放对应的端口

```bash
sudo firewall-cmd --add-port=10050/tcp  --permanent 
sudo firewall-cmd --reload 

```

源码安装官方文档 ：[点我](https://www.zabbix.com/documentation/4.0/zh/manual/installation/install)，官方文档可能没有我这里详细哦。