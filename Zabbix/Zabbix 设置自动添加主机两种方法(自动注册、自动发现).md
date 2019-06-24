在实际生产环境中，我们可能需要将很多台主机添加到 Zabbix Server 里，我们进行手动添加的话，会比较麻烦、费时，而且还容易出错。所以一般我们会设置主机自动注册。这样就比较方便。

官方文档链接 ： [点我](https://www.zabbix.com/documentation/4.0/zh/manual/discovery/auto_registration)

#### 针对zabbix agent 设置参数做下特别说明

- `Server`

  指定可以连接本 agent 的 `Zabbix Server` 或者  `Zabbix Proxy` 的 IP 。

- `ServerActive` 

   参数是用于在 自动注册和 主动监控(监控项)用的参数，设置为 `zabbix server`  或者是 `zabbix proxy`  的 IP。

- `Hostname`

  我们需要设置 Hostname ，因为我们将在 动作中的触发条件中使用，如果你没有在`zabbix_agentd.conf`中特别定义*Hostname*, 则服务器将使用agent的系统主机名命名主机。Linux中的系统主机名可以通过运行`hostname`命令获取。最后成功添加的主机名称也是该选项设置的值。



###  一、  通过 agent 自动注册到 zabbix server (官方)

**划重点：发起点就是： zabbix-agent**

**涉及配置：配置---》动作--》自动注册** 



主机自动注册配置涉及两块：

- agent 配置
- 动作-自动注册

#### 一、 agent 配置

需要配置的参数

- `Server` ： 配置为  `Zabbix Server` 或者  `Zabbix Proxy` 的 IP。

-  `ServerActive` ：配置为  `Zabbix Server` 或者  `Zabbix Proxy` 的 IP。如果端口改变了，需要在后面加上端口。

- `Hostname`：设置主机的名称。

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





### 二、通过 zabbix server 自动发现来添加主机

 **划重点：发起点就是： zabbix server** 

**涉及配置：配置---》动作--》自动发现，配置---》自动发现 **

####  zabbix agent 配置

由于发起点是 `zabbix server`,所以我们在配置参数时，只需要配置 `Server`和 `Hostname`,然后将 `ServerActive`参数注释。如果我们没有注释该参数，则又会进行自动注册了。如果我们没有设置自动注册项的话，该参数不注释也可以的。

#### Zabbix server 配置

设置自动发现规则

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E8%87%AA%E5%8A%A8%E5%8F%91%E7%8E%B0_%E6%B7%BB%E5%8A%A0%E4%B8%BB%E6%9C%BA.png)

设置动作-自动发现-创建动作

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E5%8A%A8%E4%BD%9C_%E8%87%AA%E5%8A%A8%E5%8F%91%E7%8E%B0_%E5%88%9B%E5%BB%BA%E5%8A%A8%E4%BD%9C.png)

设置动作的触发条件，就是匹配我们自动发现出来的主机，当自动发现的主机符合触发条件，就添加到 指定的主机组和链接到指定的模板。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E5%8A%A8%E4%BD%9C_%E8%87%AA%E5%8A%A8%E5%8F%91%E7%8E%B0_%E5%8A%A8%E4%BD%9C_%E8%A7%A6%E5%8F%91%E5%99%A8.png)

设置操作，链接模板，添加到主机群组。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E5%8A%A8%E4%BD%9C_%E8%87%AA%E5%8A%A8%E5%8F%91%E7%8E%B0_%E5%88%9B%E5%BB%BA%E6%93%8D%E4%BD%9C.png)

> 我们可以在 `监测---》自动发现`查看到我们自动发现到的主机。

### 三 脑洞大开

#### 脑洞大开一

在思考这个场景的过程中，我想过当 `zabbix agent` 没有固定ip(公司内部服务器)，我们该如何监控，我想可以通过让该主机自动注册到 `zabbix server`，然后使用 主动发送模式，也就是由 `zabbix-agent` 自动发送监控数据到 `zabbix server`,记住我们这里需要设置所有的监控项类型为 `zabbix agent(主动式)`。 问题点在于： **当客户端IP 变了，zabbix server 是重新添加一个新的 host，还是会自动识别  **，经过测试，发现 zabbix server 不会添加新的主机，也不会更改 之前主机的IP,但是数据是正常采集的，zabbix agent 是会有一个报错。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_agent_error.png)



#### 脑洞大开二

当把 `zabbix server` 放置在内网，没有固定ip，那么是否可以实现监控呢？我思考了下，是发现不可以的，原因有一点，就是 既然 zabbix server 没有固定ip，所以采用的模式是被动，那么在 zabbix -agent 要设置一个 `Server` 参数，这个参数的意义是允许哪个 ip 连接我的 agent 的。但是我们的 zabbix server 有没有固定 IP。所以方法是行不通的。

