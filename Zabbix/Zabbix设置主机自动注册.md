在实际生产环境中，我们可能需要将很多台主机添加到 Zabbix Server 里，我们进行手动添加的话，会比较麻烦、费时，而且还容易出错。所以一般我们会设置主机自动注册。这样就比较方便。

官方文档链接 ： [点我](https://www.zabbix.com/documentation/4.0/zh/manual/discovery/auto_registration)

主机自动注册配置涉及两块：

- agent 配置
- 动作-自动注册



#### 一、 agent 配置

- `Server`

  指定可以连接本 agent 的 `Zabbix Server` 或者  `Zabbix Proxy` 的 IP 。

-  `ServerActive` 

  我们需要在 agent 的配置文件中，指定了参数 `ServerActive` 为 `zabbix server`  或者是 `zabbix proxy`  的 IP。

- `Hostname`

  我们需要设置 Hostname ，因为我们将在 动作中的触发条件中使用，如果你没有在`zabbix_agentd.conf`中特别定义*Hostname*, 则服务器将使用agent的系统主机名命名主机。Linux中的系统主机名可以通过运行`hostname`命令获取。最后成功添加的主机名称也是该选项设置的值。

我们也可以使用其它参数值进行设置然后在触发条件中，例如 `HostMetadata` 和  `HostMetadataItem`

#### 二、动作-自动注册

配置 ---》 动作  ----》 自动注册  ---》 创建动作。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E5%8A%A8%E4%BD%9C_%E8%87%AA%E5%8A%A8%E6%B3%A8%E5%86%8C.png)



动作需要设置触发条件，我们可能只需要将自动发现的符合某个条件主机添加到某个主机群组。所以我们这里需要设置触发条件。可以通过 主机名称、主机元数据、`zabbix proxy` 来设置触发条件。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E5%8A%A8%E4%BD%9C_%E8%A7%A6%E5%8F%91%E6%9D%A1%E4%BB%B6.png)

**操作：**也就是自动发现的主机符合前面设置条件后需要设置的操作，比如添加到某个群组，链接到某个模板等等。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E5%8A%A8%E4%BD%9C_%E8%87%AA%E5%8A%A8%E6%B3%A8%E5%86%8C.png)



####  注意事项

- 如果我们设置好了，上面的配置后，但是发现没有主机注册，我们可以看看是不是我们 `zabbix server` 或者 `zabbix proxy `的监听的端口在防火墙(或者是安全组)有没有开放。默认监听端口是 10051。

- 如果我们可以看到主机注册成功了，但是 agent 的状态一直不是活跃的，那么我们可以看看我们 `zabbix agent` 的监听的端口在防火墙(或者是安全组)没有开放。默认监听端口是 10050。

