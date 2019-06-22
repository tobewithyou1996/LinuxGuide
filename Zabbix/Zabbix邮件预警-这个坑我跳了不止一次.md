每每碰到 `Zabbix`,我发现配置邮件预警这个坑，我必须要跳进去，跟它是有八辈子的仇哦，哎，接下来数数这些坑。看看你遇到过类似的吗?

#### Zabbix 预警配置流程



监控项---》触发器---》动作---》报警媒介类型---》用户报警媒介。



### 第一坑   云服务器厂商封25端口之坑

阿里云、腾讯云这些服务器厂商都限制使用 TCP 25 端口连接外部地址。如果我们一直使用 25 端口进行连接的话，你是永远发送不出邮件的，虽然说是可以解封，但是 99.9% 是解封不成功的，如果你解封成功了，可能是老天实在是看不下去了，让阿里工作人员犯晕给你解封了。

**跳坑：** 坑是自己掉进去的，爬也要爬出来，使用 465 端口，前提是你的邮件服务器开启了绑定 465 端口。

那么如果我们使用的不知名的服务器厂商，我不知道 25 端口是否被封了，我们可以使用 `telnet`，测试下。

示例：

```bash
[root@iZwz9cdow8llyjlb9lglu4Z ~]# telnet  smtp.qq.com 25
Trying 14.18.245.164...
telnet: connect to address 14.18.245.164: Connection timed out
[root@iZwz9cdow8llyjlb9lglu4Z ~]# telnet  smtp.163.com  25
Trying 220.181.12.13...
telnet: connect to address 220.181.12.13: Connection timed out
[root@iZwz9cdow8llyjlb9lglu4Z ~]# telnet  smtp.qq.com  465 
Trying 14.17.57.241...
Connected to smtp.qq.com.
Escape character is '^]'.
[root@iZwz9cdow8llyjlb9lglu4Z ~]# telnet  smtp.163.com  465
Trying 220.181.12.13...
Connected to smtp.163.com.
Escape character is '^]'.
```

阿里云有些比较早创建的 `ECS` 是没有限制25端口的，我们是可以使用25端口的。



###  第二坑 报警媒介配置之坑

有的时候我们常常忘记配置报警媒介类型，然后我们就进行预警，但是我们这里并不是讲你是否配置了报警媒介，而是讲邮件配置。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_email.png)

我们这里将示例的是两个配置。

#### **一、QQ 邮箱**

打开 `QQ` 邮箱，点击账户，选择**POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务**。 开启 `POP3/SMTP` 服务，并生成授权码。我们这获取到的授权码是 `fixleucazfkrbadf`。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/QQ_SMTP.png)

然后我们在 `Zabbix`  报警媒介类型，配置Email。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/QQ_email.png)

| 名称            | 解释                                                         | 值                                        |
| --------------- | ------------------------------------------------------------ | ----------------------------------------- |
| SMTP服务器      | 设置SMTP服务器来处理传出的消息.一般组成是 smtp + 域名        | smtp.qq.com                               |
| SMTP服务器端口  | 设置SMTP服务器端口来处理传出的消息.Zabbix 3.0版本之后*支持此选项。如果我们是可以使用25的话，我们尽量使用25，因为我在使用465的时候，是报错了的 `failed to send email: Timeout was reached: Operation timed out after 40001 milliseconds with 0 out of 0 bytes received`，可能是发送比较多导致连接超时。用不了25的服务器不要又跳坑了。 | 25,465                                    |
| SMTP HELO       | 设置正确的SMTP helo值，通常是域名.                           | qq.com                                    |
| SMTP电邮        | 发送邮件的邮件地址                                           | `1120336774@qq.com`                       |
| 安全链接        | 如果需要SSL 认证就勾选，不需要则选择 无。                    | 无                                        |
| 认证            | 用户和密码                                                   | 用户和密码                                |
| 用户名称 和密码 | 用户名，不要只填个`1120336774` 啊，我前面就是填了这个，好久才跳出来。要填`1120336774@qq.com`  ，密码填入我们获取的授权码。 | 用户名：`1120336774@qq.com` 密码 12456789 |



#### **二、自己的邮件服务器**

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_uou.png)



注意事项：

> 当我们使用 QQ 邮件服务器或者163邮件服务器等，我们如果向同一邮箱发送了比较多的邮件，邮件是很容易被放到垃圾邮箱的。而且当我们使用的是自己的邮件服务器，我们发送比较多的邮件到 QQ 邮件服务器时，我们的邮件服务器会比较容易被 QQ 邮箱标记为 垃圾邮件服务器并加入黑名单。

[发件邮箱配置官方文档](https://www.zabbix.com/documentation/4.0/zh/manual/config/notifications/media/email)

### 第三坑 动作和用户报警媒介之坑

一般我们都会创建好，监控项和触发器，但是我们一般会忘记配置动作和用户报警媒介。

#### **动作：**

一个动作由操作(例如发出通知)和条件(*什么时间*进行操作)组成，动作包含 触发动作的条件、触发动作后的操作、恢复操作、更新操作。

我们一般通过触发器警示度来配置动作，当触发器警示度大于等于警告就发邮件。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_action.png)



>  具体的动作日志，我们可以在*报表（Reports） → 动作日志（Action log）*中查看。

####  用户报警媒介

当我们需要将不同的严重性的邮件发送给不同的人，我们需要给每个用户配置报警媒介，当我们配置的预警方式是邮件的时候，我们需要为用户配置报警媒介。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_baojingmedia.png)

如果我们使用的是邮件预警的话，我们就使用的类型是 `Email` 然后在收件人里面填入收件邮箱。



### 第四坑 预警用户对生成事件的主机没有权限

当我们创建了一个用户，并且配置好了，报警媒介，在发生预警的时候，我们的配置的报警媒介的邮箱没有收到邮件，也排除了上面的问题，最终我们检查发现该用户没有对该主机没有权限。我们需要确认你创建的用户对生成事件的主机至少拥有读（read）权限，这样在预警时才能发送到对应用户的报警媒介。





