在实际监控中，除了官方自带的一些监控项，我们很多时候有一些定制化监控，比如特定的服务、TCP 连接状态等等，这时候就需要自定义监控项。自定义监控项的就是要通过用户自定义的参数来执行监控获取数据。本文将讲讲用户自定义参数和一个用户自定义参数的示例(监控 TCP 连接状态)。

###  一、用户自定义参数

[官方文档](https://www.zabbix.com/documentation/4.0/zh/manual/config/items/userparameters)

用户定义参数可以用来帮助用户实现通过 `Zabbix agent` 执行非 `Zabbix` 原生的 `agent check`。

在 agent 的配置文件中配置参数设置 `UserParameter` 。

一条用户自定义参数配置应当使用以下语法：

```
UserParameter=<key>,<command>
```

key 是对应监控项键值的值，command 是获取 监控项的值的命令，可以是脚本。

key 可以传递参数，形如 key[*],表示接受监控项传来的所有参数，在 command 可以使用 $@,$1, $2 等获取传入的参数。

`Zabbix agent` 执行的命令最多可以返回 512KB 的数据给 `zabbix server`。但是，请注意，最终可以存储在数据库中的文本值，在 `MySQL` 上的限制为 64KB 。

#### 用户自定义参数示例

```
UserParameter=ping,echo 1
```

代理将始终使用'ping'键为一个监控项返回'1'。

一个更复杂的例子:

```
UserParameter=mysql.ping,mysqladmin -uroot ping | grep -c alive
```

如果MySQL服务器是活动状态，代理将返回'1'，否则为0。

```
UserParameter=tcp.status[*],/bin/bash /opt/scripts/tcp_status.sh $1
```



### 二、配置 监控 TCP 连接状态

#### 配置 `zabbix agent`

更改 `zabbix agent` 配置文件

```bash
sudo echo "UserParameter=tcp.status[*],/bin/bash /opt/scripts/tcp_status.sh \$1 "  >>/etc/zabbix/zabbix_agentd.conf
```

下载监控脚本

```bash
# 创建目录
sudo mkdir /opt/scripts/
# 下载脚本，该链接有时间期限，github地址：https://github.com/tobewithyou1996/LinuxGuide/tree/master/Shell
sudo wget   'https://djxlsp.oss-cn-shenzhen.aliyuncs.com/shell/tcp-status.sh?OSSAccessKeyId=LTAI8hlsoWKOIPS8&Expires=1561404848&Signature=Si3RT4GdkEVKHrIgR7UaayPYcdU%3D' -O /opt/scripts/tcp_status.sh
# 更改脚本的所有者和所属组
sudo chown  zabbix:zabbix  /opt/scripts/tcp_status.sh
# 更改脚本的权限
sudo chmod  744 /opt/scripts/tcp_status.sh
### 重启zabbix agent
# centos6
sudo service zabbix-agent restart  
# centos7
sudo systemctl  restart  zabbix-agent
```

#### 导入监控模板

`xml` 文件： https://github.com/tobewithyou1996/LinuxGuide/blob/master/Zabbix/zabbix-tcp-connection.xml

选择导入该模板，该模板包含 一个应用集、11个监控项、1个触发器、一个图形。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_tcp_%E7%9B%91%E6%8E%A7_%E6%A8%A1%E6%9D%BF.png)

然后将该模板链接到主机，这样就可以监控对应的tcp连接状态数据。

**监控数据**

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_mointer_tcp_data.png)

**监控图表**

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_mointer_tcp_pic.png)

