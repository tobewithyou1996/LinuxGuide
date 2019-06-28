

当我们在线上使用了`ActiveMQ` 后，我们需要对一些参数进行监控，比如 消息是否有阻塞，哪个消息队列阻塞了，总的消息数是多少等等。下面我们就通过 `Zabbix` 结合 `Python` 脚本来实现对 `ActiveMQ`的监控。



### 一、创建 Activemq Python 监控脚本



因为 `CentOS` 系统默认安装的是 `Python2.7`，为了避免麻烦，我们这里的脚本也是对应的 `Python2`

Python2 监控脚本

```python
# -*- coding: utf-8 -*-
# @Time    : 2019/6/25 9:26
# @Author  : djx
# @Email   : djxlsp@163.com
# @File    : mointer_mq_python2.py
# @Software: PyCharm
# @Python_version: python2.7

import base64
import urllib2
import json
import logging
import sys


def activemq_mointer(userinfo_encode):
    # 总的消息阻塞数
    pending_queue_sum = 0
    # 阻塞消息的队列名称
    pending_queue_lists = ''
    # 总的消息数
    mq_sum = 0
    headers = {
        'Authorization': 'Basic {}'.format(userinfo_encode),
        'ua': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.125 Safari/537.36'
    }
    url = 'http://' + ip + ':' + port + \
          '/api/jolokia/read/org.apache.activemq:type=Broker,brokerName=localhost/Queues/'
    request = urllib2.Request(url=url, headers=headers)
    try:
        response = urllib2.urlopen(request)
    except Exception as e:
        logging.error(e)
        return {'pending_queue_sum': 110, 'pending_queue_lists': '110', 'mq_sum': 0}  # 当服务不可用时，返回预警数字，用于预警。
    activemq_info = response.read()
    activemq_info_json = json.loads(activemq_info)
    activemq_queues = activemq_info_json['value']
    for i in activemq_queues:
        queue_url = 'http://' + ip + ':' + port + \
            '/api/jolokia/read/' + i['objectName']
        queue_request = urllib2.Request(url=queue_url, headers=headers)
        try:
            queue_response = urllib2.urlopen(queue_request)
        except Exception as e:
            logging.error(e)
            return {'pending_queue_sum': 110, 'pending_queue_lists': '110', 'mq_sum': 0}
        queue_info = queue_response.read()
        info_dict = json.loads(queue_info)
        mq_sum += info_dict['value']['EnqueueCount']
        if int(info_dict['value']['QueueSize']
               ) > 0:  # 取值 QueueSize ，就是未消费的消息数量
            pending_queue_sum += info_dict['value']['QueueSize']
            pending_queue_lists += info_dict['value']['Name']
            pending_queue_lists += ' and '
            logging.info(
                "消息队列--{}--有阻塞消息--{} 条".format(
                    info_dict['value']['Name'],
                    info_dict['value']['QueueSize']))
    return {'pending_queue_sum': pending_queue_sum, 'pending_queue_lists': pending_queue_lists, 'mq_sum': mq_sum}


if __name__ == '__main__':
    # ActiveMQ 服务器信息
    username = 'admin'
    password = 'admin'
    ip = '127.0.0.1'
    port = '8161'
    userinfo = username + ':' + password
    userinfo_encode = base64.b64encode(userinfo.encode('utf8'))
    # 日志配置,注意下面日志文件的路径是采用相对路径的。
    logging.basicConfig(
        filename="/var/log/activemq_mointer.log",
        filemode="a",
        format="%(asctime)s %(name)s:%(levelname)s:%(message)s",
        datefmt="%d-%M-%Y %H:%M:%S",
        level=logging.DEBUG)
    if len(sys.argv) == 2:
        mointer_argv = sys.argv[1]
        if mointer_argv in ('pending', 'pending_lists', 'queue_sum'):
            mq_re = activemq_mointer(userinfo_encode)
            if mointer_argv == 'pending':
                print(mq_re['pending_queue_sum'])
            elif mointer_argv == 'pending_lists':
                print(mq_re['pending_queue_lists'])
            else:
                print(mq_re['mq_sum'])
        else:
            # 错误提示
            print("Please enter the correct parameters pending|pending_lists|queue_sum")
    else:
        # 错误提示
        print("Please enter the correct parameters pending|pending_lists|queue_sum")

```

使用该脚本注意事项：

1. 传入参数只能一个 ，而且只能是 `pending`, `pending_lists`, `queue_sum` ，分别代表阻塞消息数、阻塞消息队列名称、总的消息数。

2. 脚本有日志记录和异常记录，注意设置 日志文件路径，假设脚本路径位于 `/opt/scripts/`,我们在该目录下进行执行脚本的话，`activemq_mointer.log` 日志文件也就会产生在当前目录下。我们可以在路径中通过绝对路径来指定文件夹 形如 `/var/log/activemq_mointer.log`

3. 该脚本是由 `zabbix agent` 进行使用 ，所以我们需要设置该 脚本的权限，以及保证该脚本的用户有创建日志文件的权限(或者我们先前创建好对应权限日志文件)

   ```bash
   sudo chown  zabbix:zabbix  mointer_mq_python2.py
   sudo  chmod 744 mointer_mq_python2.py
   sudo  touch /var/log/activemq_mointer.log
   sudo chown  zabbix:zabbix  /var/log/activemq_mointer.log
   ```

`Python3` 脚本

```python
# -*- coding: utf-8 -*-
# @Time    : 2019/6/25 9:20
# @Author  : djx
# @Email   : djxlsp@163.com
# @File    : mointer_mq.py.py
# @Software: PyCharm
# @Python_version: python3

import  base64
import  requests
import  logging
import  sys

def activemq_mointer(userinfo_encode):
    # 总的消息阻塞数
    pending_queue_sum = 0
    # 阻塞消息的队列名称
    pending_queue_lists = ''
    # 总的消息数
    mq_sum = 0
    headers = {
        'Authorization': 'Basic {}'.format(str(userinfo_encode,'utf-8')),
        'ua': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.125 Safari/537.36'
    }
    url = 'http://' + ip + ':' + port + '/api/jolokia/read/org.apache.activemq:type=Broker,brokerName=localhost/Queues/'
    try:
        response = requests.get(url=url, headers=headers)
    except Exception as e:
        logging.error(e)
        return {'pending_queue_sum': 110, 'pending_queue_lists': '110', 'mq_sum': 0}  # 当服务不可用时，返回预警数字，用于预警。
    activemq_info = response.json()
    activemq_queues = activemq_info['value']
    for i in activemq_queues:
        queue_url = 'http://' + ip + ':' + port + '/api/jolokia/read/' + i['objectName']
        try:
            queue_info = requests.get(url=queue_url, headers=headers)
        except Exception as e:
            logging.error(e)
            return {'pending_queue_sum': 110, 'pending_queue_lists': '110', 'mq_sum': 0}
        info_dict = queue_info.json()
        mq_sum += info_dict['value']['EnqueueCount']
        if int(info_dict['value']['QueueSize']) > 0:  # 取值 QueueSize ，就是未消费的消息数量
            pending_queue_sum += info_dict['value']['QueueSize']
            pending_queue_lists += info_dict['value']['Name']
            pending_queue_lists += ' and '
            logging.info(
                "Queues--{}--peding msg --{}".format(
                    info_dict['value']['Name'],
                    info_dict['value']['QueueSize']))
    return {'pending_queue_sum': pending_queue_sum, 'pending_queue_lists': pending_queue_lists, 'mq_sum': mq_sum}

if __name__ == '__main__':
    # ActiveMQ 服务器信息
    username = 'admin'
    password = 'admin'
    ip = '127.0.0.1'
    port = '8161'
    userinfo = username + ':' + password
    userinfo_encode = base64.b64encode(userinfo.encode('utf8'))
    # 日志配置
    logging.basicConfig(
        filename="/var/log/activemq_mointer.log",
        filemode="a",
        format="%(asctime)s %(name)s:%(levelname)s:%(message)s",
        datefmt="%Y-%m-%d %H:%M:%S",
        level=logging.INFO)
    if len(sys.argv) == 2:
        mointer_argv = sys.argv[1]
        if mointer_argv in ('pending', 'pending_lists', 'queue_sum'):
            mq_re = activemq_mointer(userinfo_encode)
            if mointer_argv == 'pending':
                print(mq_re['pending_queue_sum'])
            elif mointer_argv == 'pending_lists':
                print(mq_re['pending_queue_lists'])
            else:
                print(mq_re['mq_sum'])
        else:
            # 错误提示
            print("Please enter the correct parameters pending|pending_lists|queue_sum")
    else:
        # 错误提示
        print("Please enter the correct parameters pending|pending_lists|queue_sum")

```



### 二 、设置 zabbix agent

设置 zabbix agent 

```bash
# 将监控项配置写入配置文件
sudo echo "UserParameter=activemq.mointer[*],python /opt/scripts/mointer_mq_python2.py \$1 " >> /opt/zabbix-agent/etc/zabbix_agentd.conf
# 重启zabbix agent
sudo systemctl restart  zabbix-agent 
```

### 三、导入监控项：

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_activemq_mointer.png)

监控模板 xml 文件。(该监控模板包含三个监控项，一个触发器)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<zabbix_export>
    <version>4.0</version>
    <date>2019-06-26T03:49:47Z</date>
    <groups>
        <group>
            <name>AWS-1688</name>
        </group>
        <group>
            <name>Fy-hbg</name>
        </group>
    </groups>
    <templates>
        <template>
            <template>Template App ActiveMQ</template>
            <name>Template App ActiveMQ</name>
            <description/>
            <groups>
                <group>
                    <name>AWS-1688</name>
                </group>
                <group>
                    <name>Fy-hbg</name>
                </group>
            </groups>
            <applications>
                <application>
                    <name>ActiveMQ</name>
                </application>
            </applications>
            <items>
                <item>
                    <name>activemq pending amount</name>
                    <type>0</type>
                    <snmp_community/>
                    <snmp_oid/>
                    <key>activemq.mointer[pending]</key>
                    <delay>1m</delay>
                    <history>90d</history>
                    <trends>365d</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units>条</units>
                    <snmpv3_contextname/>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authprotocol>0</snmpv3_authprotocol>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privprotocol>0</snmpv3_privprotocol>
                    <snmpv3_privpassphrase/>
                    <params/>
                    <ipmi_sensor/>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications>
                        <application>
                            <name>ActiveMQ</name>
                        </application>
                    </applications>
                    <valuemap/>
                    <logtimefmt/>
                    <preprocessing/>
                    <jmx_endpoint/>
                    <timeout>3s</timeout>
                    <url/>
                    <query_fields/>
                    <posts/>
                    <status_codes>200</status_codes>
                    <follow_redirects>1</follow_redirects>
                    <post_type>0</post_type>
                    <http_proxy/>
                    <headers/>
                    <retrieve_mode>0</retrieve_mode>
                    <request_method>0</request_method>
                    <output_format>0</output_format>
                    <allow_traps>0</allow_traps>
                    <ssl_cert_file/>
                    <ssl_key_file/>
                    <ssl_key_password/>
                    <verify_peer>0</verify_peer>
                    <verify_host>0</verify_host>
                    <master_item/>
                </item>
                <item>
                    <name>activemq pending queue name</name>
                    <type>0</type>
                    <snmp_community/>
                    <snmp_oid/>
                    <key>activemq.mointer[pending_lists]</key>
                    <delay>1m</delay>
                    <history>90d</history>
                    <trends>0</trends>
                    <status>0</status>
                    <value_type>1</value_type>
                    <allowed_hosts/>
                    <units/>
                    <snmpv3_contextname/>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authprotocol>0</snmpv3_authprotocol>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privprotocol>0</snmpv3_privprotocol>
                    <snmpv3_privpassphrase/>
                    <params/>
                    <ipmi_sensor/>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications>
                        <application>
                            <name>ActiveMQ</name>
                        </application>
                    </applications>
                    <valuemap/>
                    <logtimefmt/>
                    <preprocessing/>
                    <jmx_endpoint/>
                    <timeout>3s</timeout>
                    <url/>
                    <query_fields/>
                    <posts/>
                    <status_codes>200</status_codes>
                    <follow_redirects>1</follow_redirects>
                    <post_type>0</post_type>
                    <http_proxy/>
                    <headers/>
                    <retrieve_mode>0</retrieve_mode>
                    <request_method>0</request_method>
                    <output_format>0</output_format>
                    <allow_traps>0</allow_traps>
                    <ssl_cert_file/>
                    <ssl_key_file/>
                    <ssl_key_password/>
                    <verify_peer>0</verify_peer>
                    <verify_host>0</verify_host>
                    <master_item/>
                </item>
                <item>
                    <name>Total number of  activemq msg</name>
                    <type>0</type>
                    <snmp_community/>
                    <snmp_oid/>
                    <key>activemq.mointer[queue_sum]</key>
                    <delay>1m</delay>
                    <history>90d</history>
                    <trends>365d</trends>
                    <status>0</status>
                    <value_type>3</value_type>
                    <allowed_hosts/>
                    <units>条</units>
                    <snmpv3_contextname/>
                    <snmpv3_securityname/>
                    <snmpv3_securitylevel>0</snmpv3_securitylevel>
                    <snmpv3_authprotocol>0</snmpv3_authprotocol>
                    <snmpv3_authpassphrase/>
                    <snmpv3_privprotocol>0</snmpv3_privprotocol>
                    <snmpv3_privpassphrase/>
                    <params/>
                    <ipmi_sensor/>
                    <authtype>0</authtype>
                    <username/>
                    <password/>
                    <publickey/>
                    <privatekey/>
                    <port/>
                    <description/>
                    <inventory_link>0</inventory_link>
                    <applications>
                        <application>
                            <name>ActiveMQ</name>
                        </application>
                    </applications>
                    <valuemap/>
                    <logtimefmt/>
                    <preprocessing/>
                    <jmx_endpoint/>
                    <timeout>3s</timeout>
                    <url/>
                    <query_fields/>
                    <posts/>
                    <status_codes>200</status_codes>
                    <follow_redirects>1</follow_redirects>
                    <post_type>0</post_type>
                    <http_proxy/>
                    <headers/>
                    <retrieve_mode>0</retrieve_mode>
                    <request_method>0</request_method>
                    <output_format>0</output_format>
                    <allow_traps>0</allow_traps>
                    <ssl_cert_file/>
                    <ssl_key_file/>
                    <ssl_key_password/>
                    <verify_peer>0</verify_peer>
                    <verify_host>0</verify_host>
                    <master_item/>
                </item>
            </items>
            <discovery_rules/>
            <httptests/>
            <macros/>
            <templates/>
            <screens/>
        </template>
    </templates>
    <triggers>
        <trigger>
            <expression>{Template App ActiveMQ:activemq.mointer[pending].avg(10m)}&gt;=5</expression>
            <recovery_mode>1</recovery_mode>
            <recovery_expression>{Template App ActiveMQ:activemq.mointer[pending].avg(5m)}=0</recovery_expression>
            <name>activemq queue  pending on {HOST.NAME}</name>
            <correlation_mode>0</correlation_mode>
            <correlation_tag/>
            <url/>
            <status>0</status>
            <priority>3</priority>
            <description>activemq 消息发生阻塞，10分钟内平均阻塞消息数超过5条</description>
            <type>0</type>
            <manual_close>0</manual_close>
            <dependencies/>
            <tags/>
        </trigger>
    </triggers>
</zabbix_export>

```

将该监控模板链接到对应的主机。

我们可以看到我们监控的数据了。

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_activemq_mointer_data.png)

至此，ActiveMQ 的监控项都已经配置好了。