##  Zabbix 集成 睿象云智能告警平台 CA ( Cloud Alert )

### 一 、简介与前期了解

Cloud Alert 通过应用，接入监控系统/平台的告警，集中管理您的告警，统一分派通知，统一分析。

这个平台最先了解和使用是在 2017 年下半年，之前的名称叫 `oneitsM`。预警产品名称为 ： `OneAlert`, 现在该产品已经迁移到 睿象云，并更名为 `CloudAlert` 。本文主要是介绍和记录下该预警产品的使用。

我们首先要注册一个账号：[官网链接](https://aiops.com/CAIntroduce.html)，然后登陆我们的账号。选择我们的 `Cloud Alert`。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E9%A2%84%E8%AD%A6_Cloud_Alert.png)

然后点击我们上方的集成。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_%E9%A2%84%E8%AD%A6_Cloud_Alert_%E9%9B%86%E6%88%90.png)

我们可以直接集成业界主流的监控工具，如：Zabbix、Nagios、Prometheus、OpenFalcon、SolarWinds等，同时也可以通过 Email 邮箱集成或者 REST API 方式接入您的告警。

###  二、集成到 Zabbix

我们需要先获取我们的 `APPkey` ，然后在安装的时候，传入该参数。

一、安装 Agent

1. 切换到 `zabbix` 脚本目录 (如何查看 `zabbix` 脚本目录)：

```bash
cd /usr/local/zabbix-server/share/zabbix/alertscripts 
```

2. 获取Cloud Alert Agent包：

```bash
wget https://download.aiops.com/ca_agent/zabbix/ca_zabbix_release-2.1.0.tar.gz
```

3. 解压、安装。

```bash
tar -xzf ca_zabbix_release-2.1.0.tar.gz 
cd cloudalert/bin 
bash install.sh APPkey
```

> 注：1、在安装过程中根据安装提示，输入zabbix管理地址、管理员用户名、密码。
>
> ​		 2、zabbix管理地址正确示例：http://zabbix.server.com/zabbix

4. 当提示"安装成功"时表示安装成功！
5. 在 `zabbix server` 管理界面查看是否添加成功。

![1561622519970](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_cloud_alert_web.png)

出现该脚本就意味着成功了。

### 三 、创建分派策略和通知策略

这里要严重的注意一点,就是 `CloudAlert` 的 预警级别只有三种,它和 `zabbix` 的级别对应见下表.我们在设置通知策略和分派策略需要注意.

| zabbix 级别状态         | 参数值 | OneAlert 级别状态 |
| :---------------------- | :----- | :---------------- |
| information (信息)      | 1      | 提醒              |
| not_classified (未分类) | 1      | 提醒              |
| warning (警告)          | 2      | 警告              |
| average (一般严重)      | 2      | 警告              |
| high (严重)             | 3      | 严重              |
| disaster (灾难)         | 3      | 严重              |

#### 分派策略

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_Cloud_Alert%E5%88%9B%E5%BB%BA%E5%88%86%E6%B4%BE%E7%AD%96%E7%95%A5.png)



#### 通知策略

我们这里只使用到 `CloudAlert` 的通知策略的通知方式中的电话和短信,因为我们 微信已经对接了我们的企业微信预警,邮箱也使用了企业邮箱(进垃圾邮箱的概率更低一些).所以我们这里只设置 通知方式为 电话和短信. 并且只有在严重预警的时候才会触发.

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_CloudAlert_%E9%80%9A%E7%9F%A5%E7%AD%96%E7%95%A5.png)



### 四 、设置动作

在执行安装脚本的时候,默认已经帮我们添加好了动作 `cloudalert action`。但是没有设置触发条件,我们可以设置一下触发条件,触发器示警度 大于等于 **严重** 的时候,进行触发该动作.



![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_cloudalert_action.png)



### 五 、生成预警

当我们的预警达到阈值,就会触发报警.

短信预警内容:

```
【睿象云】16:48:53,发生严重级别告警:Zabbix agent on Test-186 is unreachable for 5 minutesTest-186 Agent ping:Up (1)Zabbix agent on Test-186 is unreachable for 5 minutes,告警对象:Test-186,告警编号:585679
```



### 六 、卸载Cloud Alert

#### Web 设置卸载

- 删除报警媒介 `cloudalert media`。

- 删除用户群组  `cloudalert group`。

- 删除用户  `cloudalert`。

- 删除动作  `cloudalert action`。

> 有人会说我们没有添加上面的东西，为什么会存在上面这些内容，我们在 执行 `install.sh`  脚本的时候就会添加这些.

#### 脚本文件卸载

​	删除脚本   删除  `/usr/local/zabbix-server/share/zabbix/alertscripts ` 的`cloudalert`文件夹。



### 七、注意事项

#### 错误内容

我安装的  `zabbix server ` 是使用的 docker 安装的, 脚本目录是使用的数据卷(我们可以用 `docker inspect container_id  ` 查看到)。我们将脚本放置在该数据卷后，安装，也提示成功了，但是在预警的时候，有报错。

报错内容如下：

```bash
/usr/lib/zabbix/alertscripts/cloudalert/bin
/usr/lib/zabbix/alertscripts/cloudalert/bin/log.sh: line 11: /var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert/logs/cloudalert.log: No such file or directory
cp: cannot stat '/var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert/logs/cloudalert.log': No such file or directory
/usr/lib/zabbix/alertscripts/cloudalert/bin/log.sh: line 16: /var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert/logs/cloudalert.log: No such file or directory
% Total % Received % Xferd Average Speed Time Time Time Current
Dload Upload Total Spent Left Speed

0 0 0 0 0 0 0 0 --:--:-- --:--:-- --:--:-- 0
100 623 0 89 100 534 1141 6850 --:--:-- --:--:-- --:--:-- 6935
/usr/lib/zabbix/alertscripts/cloudalert/bin/log.sh: line 11: /var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert/logs/cloudalert.log: No such file or directory
cp: cannot stat '/var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert/logs/cloudalert.log': No such file or directory
/usr/lib/zabbix/alertscripts/cloudalert/bin/log.sh: line 16: /var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert/logs/cloudalert.log: No such file or directory
```

####  解析过程与解决问题

从上面我们可以看到是脚本 `log.sh` 执行的过程中报错了，提示没有该文件，它写入的文件是 `/var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert/logs/cloudalert.log` ，这个文件路径是 docker 宿主机的日志文件路径，程序在 docker 里面运行，这个路径肯定是获取不到的。我们通过查看 `log.sh` 脚本发现,

```bash
#!/bin/bash
if [ -z "$DIR" ]; then
    DIR="$( cd "$( dirname "$0"  )" && pwd  )"
fi

source $DIR/cloudalert.conf
function log(){
  path=$base_path
  log=$path/logs/cloudalert.log
  time=`date +%Y-%m-%d\ %H:%M:%S`
  echo $time $1 [$2]: "$3" >> $log
  bak_log=$path/logs/cloudalert.log_`date +%Y-%m`
  if [ ! -f $bak_log ];
  then
    cp $log $bak_log
    > $log
  fi
}
```

日志路径引用了  `base_path`,这个值是在  `cloudalert.conf` 里定义的. 我们可以在配置文件中看到, `base_path` 值获取的是 宿主机  `cloudalert` 所在的路径. 而不是实际在 docker 容器里的路径。

```

current_path=/var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert/bin
base_path=/var/lib/docker/volumes/bb221b74a7d8ad528194867830db0c1ac8fdc31f2ab0ee4456ffce61646fd83a/_data/cloudalert
agentVersion=1130
AppKey=1233444555666
zabbix_host=http://127.0.0.1
zabbix_url=http://127.0.0.1/api_jsonrpc.php
user=Admin
password=admin            
```



我们将 `base_path` 的值更改为 在docker 容器里面的值 `/usr/lib/zabbix/alertscripts/cloudalert`  .

然后就没有报错了.

####  反思

这个问题的引起，是我在 宿主机下将执行的该脚本，导致的脚本执行的时候获取的是宿主机的目录，而不是 docker 主机里面的目录路径，在 docker 容器里面执行脚本，即可避免该问题。



