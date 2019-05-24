

[TOC]

**本文所有内容基于：**

```txt
Docker-CE
Server Version: 18.09.6
Storage Driver: overlay2
Kernel Version: 3.10.0-862.el7.x86_64
Operating System: CentOS Linux 7 (Core)
```

Docker  日志分为两类：

-  Docker 引擎日志(也就是 dockerd 运行时的日志)， 
-  容器的日志，容器内的服务产生的日志。



## 一 、Docker 引擎日志

Docker 引擎日志一般是交给了 Upstart(Ubuntu 14.04) 或者 systemd (CentOS 7, Ubuntu 16.04)。前者一般位于 /var/log/upstart/docker.log 下，后者我们一般 通过  `journalctl   -u  docker `  来进行查看。



| 系统                   | 日志位置                                                     |
| ---------------------- | ------------------------------------------------------------ |
| Ubuntu(14.04)          | `/var/log/upstart/docker.log`                                |
| Ubuntu(16.04)          | `journalctl -u docker.service`                               |
| CentOS 7/RHEL 7/Fedora | `journalctl -u docker.service`                               |
| CoreOS                 | `journalctl -u docker.service`                               |
| OpenSuSE               | `journalctl -u docker.service`                               |
| OSX                    | `~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/log/d‌ocker.log` |
| Debian GNU/Linux 7     | `/var/log/daemon.log`                                        |
| Debian GNU/Linux 8     | `journalctl -u docker.service`                               |
| Boot2Docker            | `/var/log/docker.log`                                        |

以上内容来自：<https://blog.lab99.org/post/docker-2016-07-14-faq.html>

## 二、容器日志



###  2.1、常用查看日志命令——docker logs

`docker logs CONTAINER` 显示当前运行的容器的日志信息， UNIX 和 Linux 的命令有三种 输入输出，分别是 STDIN(标准输入)、STDOUT(标准输出)、STDERR(标准错误输出)，docker logs  显示的内容包含 STOUT 和 STDERR。在生产环境，如果我们的应用输出到我们的日志文件里，所以我们在使用  docker  logs 一般收集不到太多重要的日志信息。

>  - nginx 官方镜像，使用了一种方式，让日志输出到 STDOUT，也就是 创建一个符号链接 `/var/log/nginx/access.log` 到 `/dev/stdout`。
>  - httpd 使用的是 让其输出到指定文件 ，正常日志输出到 `/proc/self/fd/1` (STDOUT) ，错误日志输出到 `/proc/self/fd/2` (STDERR)。
>  - 当日志量比较大的时候，我们使用 docker logs   来查看日志，会对 docker daemon 造成比较大的压力，容器导致容器创建慢等一系列问题。
>  - **只有使用了 `local 、json-file、journald`  的日志驱动的容器才可以使用 docker logs 捕获日志，使用其他日志驱动无法使用 `docker logs`**

  

### 2.2 、Docker 日志 驱动

Docker 提供了两种模式用于将消息从容器到日志驱动。

- (默认)拒绝，阻塞从容器到容器驱动
- 非阻塞传递,日志将储存在容器的缓冲区。

> 当缓冲区满，旧的日志将被丢弃。

在 mode 日志选项控制使用 `blocking(默认)` 或者 `non-blocking`, 当设置为 `non-blocking` 需要设置 `max-buffer-size` 参数(默认为 1MB)。

支持的驱动

|                                                              | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `none`                                                       | 运行的容器没有日志，`docker logs`也不返回任何输出。          |
| [`local`](https://docs.docker.com/config/containers/logging/local/) | 日志以自定义格式存储，旨在实现最小开销。                     |
| [`json-file`](https://docs.docker.com/config/containers/logging/json-file/) | 日志格式为JSON。Docker的默认日志记录驱动程序。               |
| [`syslog`](https://docs.docker.com/config/containers/logging/syslog/) | 将日志消息写入`syslog`。该`syslog`守护程序必须在主机上运行。 |
| [`journald`](https://docs.docker.com/config/containers/logging/journald/) | 将日志消息写入`journald`。该`journald`守护程序必须在主机上运行。 |
| [`gelf`](https://docs.docker.com/config/containers/logging/gelf/) | 将日志消息写入Graylog扩展日志格式（GELF）端点，例如Graylog或Logstash。 |
| [`fluentd`](https://docs.docker.com/config/containers/logging/fluentd/) | 将日志消息写入`fluentd`（转发输入）。该`fluentd`守护程序必须在主机上运行。 |
| [`awslogs`](https://docs.docker.com/config/containers/logging/awslogs/) | 将日志消息写入Amazon CloudWatch Logs。                       |
| [`splunk`](https://docs.docker.com/config/containers/logging/splunk/) | 使用HTTP事件收集器将日志消息写入`splunk`。                   |
| [`etwlogs`](https://docs.docker.com/config/containers/logging/etwlogs/) | 将日志消息写为Windows事件跟踪（ETW）事件。仅适用于Windows平台。 |
| [`gcplogs`](https://docs.docker.com/config/containers/logging/gcplogs/) | 将日志消息写入Google Cloud Platform（GCP）Logging。          |
| [`logentries`](https://docs.docker.com/config/containers/logging/logentries/) | 将日志消息写入Rapid7 Logentries。                            |



使用 Docker-CE 版本，`docker logs `命令 仅仅适用于以下驱动程序(前面 docker logs 详解也提及到了)

- local
- json-file
- journald

![1558055133186](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/docker/docker-log-driver.png)

#### Docker 日志驱动常用命令

查看系统当前设置的日志驱动

```bash
docker  info |grep  "Logging Driver"  / docker info --format '{{.LoggingDriver}}'
```

查看单个容器的设置的日志驱动

```bash
docker inspect  -f '{{.HostConfig.LogConfig.Type}}'   容器id
```

#### Docker 日志驱动全局配置更改

修改日志驱动，在配置文件 `/etc/docker/daemon.json`（注意该文件内容是 JSON 格式的）进行配置即可。

示例：

```
{
  "log-driver": "syslog"
}
```

以上更改是针对所有的容器的日志驱动的。我们也可以单独为单一容器设置日志驱动。

#### Docker 单一容器日志驱动配置

在 运行容器的时候指定 日志驱动 ` --log-driver `。

```bash
docker  run  -itd --log-driver none alpine ash # 这里指定的日志驱动为 none 
```



#### 日志驱动 一 、local

`local`  日志驱动 记录从容器的 `STOUT/STDERR` 的输出，并写到宿主机的磁盘。

默认情况下，local  日志驱动为每个容器保留 100MB 的日志信息，并启用自动压缩来保存。(经过测试，保留100MB 的日志是指没有经过压缩的日志)

local 日志驱动的储存位置 `/var/lib/docker/containers/容器id/local-logs/` 以 `container.log` 命名。

 **local 驱动支持的选项**

| 选项       | 描述                                                         | 示例值                     |
| :--------- | :----------------------------------------------------------- | :------------------------- |
| `max-size` | 切割之前日志的最大大小。可取值为(k,m,g)， 默认为20m。        | `--log-opt max-size=10m`   |
| `max-file` | 可以存在的最大日志文件数。如果超过最大值，则会删除最旧的文件。**仅在max-size设置时有效。默认为5。 | `--log-opt max-file=3`     |
| `compress` | 对应切割日志文件是否启用压缩。默认情况下启用。               | `--log-opt compress=false` |



**全局日志驱动设置为—local**

在配置文件 `/etc/docker/daemon.json`（注意该文件内容是 JSON 格式的）进行配置即可。

```
{
  "log-driver": "local",
  "log-opts": {
    "max-size": "10m"
  }
}
```

重启 docker  即可生效。

**单个容器日志驱动设置为—local**

运行容器并设定为 `local` 驱动。

```bash
#  运行一个容器 ，并设定日志驱动为 local ，并运行命令 ping www.baidu.com
[root@localhost docker]# docker run  -itd  --log-driver  local  alpine  ping www.baidu.com 
3795b6483534961c1d5223359ad1106433ce2bf25e18b981a47a2d79ad7a3156
#  查看运行的容器的 日志驱动是否是 local
[root@localhost docker]# docker inspect  -f '{{.HostConfig.LogConfig.Type}}'   3795b6483534961c
local
# 查看日志
[root@localhost local-logs]# tail -f  /var/lib/docker/containers/3795b6483534961c1d5223359ad1106433ce2bf25e18b981a47a2d79ad7a3156/local-logs/container.log 
NNdoutםѰ͕̈:64 bytes from 14.215.177.38: seq=816 ttl=55 time=5.320 ms
NNdoutهµ͕̈͡:64 bytes from 14.215.177.38: seq=817 ttl=55 time=4.950 ms
```



> 注意事项： 经过测试，当我们产生了100 MB 大小的日志时 会有 四个压缩文件和一个 `container.log`：
>
> ```bash
> [root@localhost local-logs]# ls -l
> total 32544
> -rw-r-----. 1 root root 18339944 May 16 09:41 container.log
> -rw-r-----. 1 root root  3698660 May 16 09:41 container.log.1.gz
> -rw-r-----. 1 root root  3726315 May 16 09:41 container.log.2.gz
> -rw-r-----. 1 root root  3805668 May 16 09:41 container.log.3.gz
> -rw-r-----. 1 root root  3744104 May 16 09:41 container.log.4.gz
> ```
>
> 那么当超过了 100MB 的日志文件，日志文件会继续写入到  `container.log`，但是会将  `container.log` 日志中老的日志删除，追加新的，也就是 当写满 100MB 日志后 ，再产生一条新日志，会删除  `container.log` 中的一条老日志，保存 100MB 的大小。**这个 对我们是会有一些影响的，**
>
> ```tex
> 当我运行系统时 第一天由于bug产生了 100MB 日志，那么之前的日志就已经有 80MB 日志变成的压缩包，所以我在后续的运行中，只能获取最近的 20MB日志。
> ```



#### 日志驱动 二、 默认的日志驱动—JSON

**所有容器默认的日志驱动 `json-file`**。

`json-file` 日志驱动 记录从容器的 `STOUT/STDERR` 的输出 ，用 JSON 的格式写到文件中，日志中不仅包含着 输出日志，还有时间戳和 输出格式。下面是一个 `ping www.baidu.com`  对应的 JSON 日志

```json
{"log":"64 bytes from 14.215.177.39: seq=34 ttl=55 time=7.067 ms\r\n","stream":"stdout","time":"2019-05-16T14:14:15.030612567Z"}
```

json-file  日志的路径位于 `/var/lib/docker/containers/container_id/container_id-json.log`。

`json-file` 的 日志驱动支持以下选项：

| 选项        | 描述                                                         | 示例值                                   |
| :---------- | :----------------------------------------------------------- | :--------------------------------------- |
| `max-size`  | 切割之前日志的最大大小。可取值单位为(k,m,g)， 默认为-1（表示无限制）。 | `--log-opt max-size=10m`                 |
| `max-file`  | 可以存在的最大日志文件数。如果切割日志会创建超过阈值的文件数，则会删除最旧的文件。**仅在max-size设置时有效。**正整数。默认为1。 | `--log-opt max-file=3`                   |
| `labels`    | 适用于启动Docker守护程序时。此守护程序接受的以逗号分隔的与日志记录相关的标签列表。 | `--log-opt labels=production_status,geo` |
| `env`       | 适用于启动Docker守护程序时。此守护程序接受的以逗号分隔的与日志记录相关的环境变量列表。 | `--log-opt env=os,customer`              |
| `env-regex` | 类似于并兼容`env`。用于匹配与日志记录相关的环境变量的正则表达式。 | `--log-opt env-regex=^(os|customer).`    |
| `compress`  | 切割的日志是否进行压缩。默认是`disabled`。                   | `--log-opt compress=true`                |

**`json-file` 的日志驱动示例**

```bash
# 设置 日志驱动为 json-file ，我们也可以不设置，因为默认就是 json-file
docker run  -itd  --name  test-log-json  --log-driver json-file   alpine  ping www.baidu.com
199608b2e2c52136d2a17e539e9ef7fbacf97f1293678aded421dadbdb006a5e

# 查看日志,日志名称就是 容器名称-json.log
tail -f /var/lib/docker/containers/199608b2e2c52136d2a17e539e9ef7fbacf97f1293678aded421dadbdb006a5e/199608b2e2c52136d2a17e539e9ef7fbacf97f1293678aded421dadbdb006a5e-json.log

{"log":"64 bytes from 14.215.177.39: seq=13 ttl=55 time=15.023 ms\r\n","stream":"stdout","time":"2019-05-16T14:13:54.003118877Z"}
{"log":"64 bytes from 14.215.177.39: seq=14 ttl=55 time=9.640 ms\r\n","stream":"stdout","time":"2019-05-16T14:13:54.999011017Z"}
{"log":"64 bytes from 14.215.177.39: seq=15 ttl=55 time=8.938 ms\r\n","stream":"stdout","time":"2019-05-16T14:13:55.998612636Z"}
{"log":"64 bytes from 14.215.177.39: seq=16 ttl=55 time=18.086 ms\r\n","stream":"stdout","time":"2019-05-16T14:13:57.011235913Z"}
{"log":"64 bytes from 14.215.177.39: seq=17 ttl=55 time=12.615 ms\r\n","stream":"stdout","time":"2019-05-16T14:13:58.007104112Z"}
{"log":"64 bytes from 14.215.177.39: seq=18 ttl=55 time=11.001 ms\r\n","stream":"stdout","time":"2019-05-16T14:13:59.007559413Z"}
```

#### 日志驱动 三、syslog

syslog 日志驱动将日志路由到 syslog 服务器，syslog 以原始的字符串作为 日志消息元数据，接收方可以提取以下的消息：

- level  日志等级 ，如`debug`，`warning`，`error`，`info`。
- timestamp  时间戳
- hostname  事件发生的主机
- facillty  系统模块
- 进程名称和进程 ID  

**`syslog` 日志驱动全局配置**

编辑 `/etc/docker/daemon.json`  文件

```json
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "udp://1.2.3.4:1111"
  }
}
```

重启 docker  即可生效。

| Option                   | Description                                                  | Example value                                                |
| :----------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `syslog-address`         | 指定syslog 服务所在的服务器和使用的协议和端口。 格式：`[tcp|udp|tcp+tls]://host:port,unix://path, orunixgram://path`. 默认端口是 514. | `--log-opt syslog-address=tcp+tls://192.168.1.3:514`, `--log-opt syslog-address=unix:///tmp/syslog.sock` |
| `syslog-facility`        | 使用的 `syslog` 的设备，  具体设备名称见 [syslog documentation](https://tools.ietf.org/html/rfc5424#section-6.2.1). | `--log-opt syslog-facility=daemon`                           |
| `syslog-tls-ca-cert`     | 如果使用的是 `tcp+tls` 的地址，指定CA 证书的地址，如果没有使用，则不设置该选项。 | `--log-opt syslog-tls-ca-cert=/etc/ca-certificates/custom/ca.pem` |
| `syslog-tls-cert`        | 如果使用的是 `tcp+tls` 的地址，指定 TLS 证书的地址，如果没有使用，则不设置该选项。 | `--log-opt syslog-tls-cert=/etc/ca-certificates/custom/cert.pem` |
| `syslog-tls-key`         | 如果使用的是 `tcp+tls` 的地址，指定 TLS 证书 key的地址，如果没有使用，则不设置该选项。** | `--log-opt syslog-tls-key=/etc/ca-certificates/custom/key.pem` |
| `syslog-tls-skip-verify` | 如果设置为 true ，会跳过 TLS 验证，默认为 false              | `--log-opt syslog-tls-skip-verify=true`                      |
| `tag`                    | 将应用程序的名称附加到 `syslog` 消息中，默认情况下使用容器ID的前12位去 标记这个日志信息。 | `--log-opt tag=mailer`                                       |
| `syslog-format`          | `syslog` 使用的消息格式 如果未指定则使用本地 UNIX syslog 格式，rfc5424micro 格式具有微妙时间戳。 | `--log-opt syslog-format=rfc5424micro`                       |
| `labels`                 | 启动 docker 时，配置与日志相关的标签，以逗号分割             | `--log-opt labels=production_status,geo`                     |
| `env`                    | 启动 docker 时，指定环境变量用于日志中，以逗号分隔           | `--log-opt env=os,customer`                                  |
| `env-regex`              | 类似并兼容 `env`，                                           | `--log-opt env-regex=^(os\|customer)`                        |

**单个容器日志驱动设置为—syslog **

`Linux` 系统中 我们用的系统日志模块时  `rsyslog` ，它是基于`syslog` 的标准实现。我们要使用 syslog 驱动需要使用 系统自带的 `rsyslog` 服务。

```bash
# 查看当前 rsyslog 版本和基本信息
[root@localhost harbor]# rsyslogd  -v
rsyslogd 8.24.0, compiled with:
	PLATFORM:				x86_64-redhat-linux-gnu
	PLATFORM (lsb_release -d):		
	FEATURE_REGEXP:				Yes
	GSSAPI Kerberos 5 support:		Yes
	FEATURE_DEBUG (debug build, slow code):	No
	32bit Atomic operations supported:	Yes
	64bit Atomic operations supported:	Yes
	memory allocator:			system default
	Runtime Instrumentation (slow code):	No
	uuid support:				Yes
	Number of Bits in RainerScript integers: 64

See http://www.rsyslog.com for more information.
```

配置 syslog , 在配置文件 `/etc/rsyslog.conf`  大约14-20行，我们可以看到两个配置，一个udp，一个tcp ，都是监听 514 端口，提供 syslog 的接收。选择 tcp 就将 tcp 的两个配置的前面 # 号注释即可。

```
# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514

# Provides TCP syslog reception
#$ModLoad imtcp  
#$InputTCPServerRun 514
```

然后重启 rsyslog，我们可以看到514端口在监听。

```bash
systemctl restart  rsyslog
[root@localhost harbor]# netstat -ntul |grep 514
tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN     
tcp6       0      0 :::514                  :::*                    LISTEN  
```

启动一个以  `syslog` 为驱动的容器。

```bash
docker  run -d -it  -p 87:80 --log-driver syslog --log-opt syslog-address=tcp://127.0.0.1:514  --name nginx-syslog   nginx

```

访问并查看日志

```bash
# 访问nginx
curl 127.0.0.1:87
# 查看访问日志
tail -f  /var/log/messages
May 17 15:56:48 localhost fe18924aefde[6141]: 172.17.0.1 - - [17/May/2019:07:56:48 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"#015
May 17 15:58:16 localhost fe18924aefde[6141]: 172.17.0.1 - - [17/May/2019:07:58:16 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"#015
```



#### 日志驱动 四、Journald

 `journald` 日志驱动程序将容器的日志发送到 `systemd journal`, 可以使用 `journal API` 或者使用 `docker logs` 来查日志。



除了日志本身以外， `journald`  日志驱动还会在日志加上下面的数据与消息一起储存。

| Field                                | Description                                                  |
| :----------------------------------- | :----------------------------------------------------------- |
| `CONTAINER_ID`                       | 容器ID,为 12个字符                                           |
| `CONTAINER_ID_FULL`                  | 完整的容器ID，为64个字符                                     |
| `CONTAINER_NAME`                     | 启动时容器的名称，如果容器后面更改了名称，日志中的名称不会更改。 |
| `CONTAINER_TAG`, `SYSLOG_IDENTIFIER` | 容器的tag.                                                   |
| `CONTAINER_PARTIAL_MESSAGE`          | 当日志比较长的时候使用标记来表示(显示日志的大小)             |

选项

| 选项        | 是否必须 | 描述                                                         |
| :---------- | :------- | :----------------------------------------------------------- |
| `tag`       | 可选的   | 指定要在日志中设置`CONTAINER_TAG`和`SYSLOG_IDENTIFIER`值的模板。 |
| `labels`    | 可选的   | 以逗号分隔的标签列表，如果为容器指定了这些标签，则应包含在消息中。 |
| `env`       | 可选的   | 如果为容器指定了这些变量，则以逗号分隔的环境变量键列表（应包含在消息中）。 |
| `env-regex` | 可选的   | 与env类似并兼容。用于匹配与日志记录相关的环境变量的正则表达式 。 |



**`journald` 日志驱动全局配置**

编辑 `/etc/docker/daemon.json`  文件

```
{
  "log-driver": "journald"
}
```

**单个容器日志驱动设置为—`journald`** 

```
docker  run  -d -it --log-driver=journald \
    --log-opt labels=location \
    --log-opt env=TEST \
    --env "TEST=false" \
    --label location=china \
    --name  nginx-journald\
    -p 80:80\
    nginx
```

查看日志 `journalctl`

```bash
# 只查询指定容器的相关消息
 journalctl CONTAINER_NAME=webserver
# -b 指定从上次启动以来的所有消息
 journalctl -b CONTAINER_NAME=webserver
# -o 指定日志消息格式，-o json 表示以json 格式返回日志消息
 journalctl -o json CONTAINER_NAME=webserver
# -f 一直捕获日志输出
 journalctl -f CONTAINER_NAME=webserver
```

> 如果我们的容器在启动的时候加了 -t 参数，启用了 TTY 的话，那么我查看日志是会像下面一样
>
> ```text
> May 17 17:19:26 localhost.localdomain 2a338e4631fe[6141]: [104B blob data]
> May 17 17:19:32 localhost.localdomain 2a338e4631fe[6141]: [104B blob data]
> ```
>
> 显示`[104B blob data]` 而不是完整日志原因是因为有 `\r` 的存在，如果我们要完整显示，需要加上参数 `--all` 。



## 三、 生产环境中该如何储存容器中的日志

我们在上面看到了 Docker 官方提供了 很多日志驱动，但是上面的这些驱动都是针对的 标准输出的日志驱动。

#### 容器日志分类

容器的日志实际是有两大类的：

-  **标准输出的** ，也就是 STDOUT 、STDERR ,**这类日志我们可以通过 Docker 官方的日志驱动进行收集。**

  示例：Nginx 日志，Nginx 日志有 `access.log` 和 `error.log` ，我们在 Docker Hub 上可以看到  Nginx 的 dockerfile  对于这两个日志的处理是：

  ```dockerfile
  RUN ln -sf /dev/stdout /var/log/nginx/access.log \
  	&& ln -sf /dev/stderr /var/log/nginx/error.log
  ```

  都软连接到 `/dev/stdout` 和 `/dev/stderr`  ，也就是标准输出，所以这类 容器是可以使用 Docker 官方的日志驱动。

- **文本日志**，存在在于容器内部，并没有重定向到 容器的标准输出的日志。

  示例： Tomcat 日志，Tomcat 有 catalina、localhost、manager、admin、host-manager，我们可以在 Docker Hub 看到 Tomcat 的 dockerfile 只有对于 catalina 进行处理，其它日志将储存在容器里。

  ```dockerfile
  CMD ["catalina.sh", "run"]
  ```

  我们运行了一个 Tomcat 容器 ，然后进行访问后，并登陆到容器内部，我们可以看到产生了文本日志：

  ```bash
  root@25ba00fdab97:/usr/local/tomcat/logs# ls -l
  total 16
  -rw-r-----. 1 root root 6822 May 17 14:36 catalina.2019-05-17.log
  -rw-r-----. 1 root root    0 May 17 14:36 host-manager.2019-05-17.log
  -rw-r-----. 1 root root  459 May 17 14:36 localhost.2019-05-17.log
  -rw-r-----. 1 root root 1017 May 17 14:37 localhost_access_log.2019-05-17.txt
  -rw-r-----. 1 root root    0 May 17 14:36 manager.2019-05-17.log
  ```

  这类容器我们下面有专门的方案来应对。

### 一、当是完全是标准输出的类型的容器

我们可以选择  json-file 、syslog、local 等 Docker 支持的日志驱动。



### 二、当有文件文本日志的类型容器

#### 方案一 挂载目录  bind

创建一个目录，将目录挂载到 容器中产生日志的目录。

```bash
--mount  type=bind,src=/opt/logs/,dst=/usr/local/tomcat/logs/ 
```

示例：

```bash
# 创建挂载目录/opt/logs
[root@fy-local-2 /]# mkdir  /opt/logs
# 创建容器tomcat-bind 并将 /opt/logs 挂载至 /usr/local/tomcat/logs/
[root@fy-local-2 /]# docker  run -d  --name  tomcat-bind  -P  --mount  type=bind,src=/opt/logs/,dst=/usr/local/tomcat/logs/   tomcat 
[root@fy-local-2 /]# ls -l /opt/logs/
total 12
-rw-r----- 1 root root 6820 May 22 17:31 catalina.2019-05-22.log
-rw-r----- 1 root root    0 May 22 17:31 host-manager.2019-05-22.log
-rw-r----- 1 root root  459 May 22 17:31 localhost.2019-05-22.log
-rw-r----- 1 root root    0 May 22 17:31 localhost_access_log.2019-05-22.txt
-rw-r----- 1 root root    0 May 22 17:31 manager.2019-05-22.log
```



#### 方案二 使用数据卷 volume

创建数据卷，创建容器时绑定数据卷，

```bash
--mount  type=volume  src=volume_name  dst=/usr/local/tomcat/logs/ 
```

示例：

```bash
# 创建tomcat应用数据卷名称为 tomcat
[root@fy-local-2 /]# docker volume  create  tomcat
# 创建容器tomcat-volume 并指定数据卷为 tomcat，绑定至 /usr/local/tomcat/logs/
[root@fy-local-2 /]# docker  run -d  --name  tomcat-volume   -P  --mount  type=volume,src=tomcat,dst=/usr/local/tomcat/logs/   tomcat
# 查看数据卷里面的内容
[root@fy-local-2 /]# ls -l /var/lib/docker/volumes/tomcat/_data/
total 12
-rw-r----- 1 root root 6820 May 22 17:33 catalina.2019-05-22.log
-rw-r----- 1 root root    0 May 22 17:33 host-manager.2019-05-22.log
-rw-r----- 1 root root  459 May 22 17:33 localhost.2019-05-22.log
-rw-r----- 1 root root    0 May 22 17:33 localhost_access_log.2019-05-22.txt
-rw-r----- 1 root root    0 May 22 17:33 manager.2019-05-22.log

```

#### 方案三 计算容器 rootfs 挂载点

此方案的文字内容摘抄于 <https://yq.aliyun.com/articles/672054>

使用挂载宿主机目录的方式采集日志对应用会有一定的侵入性，因为它要求容器启动的时候包含挂载命令。如果采集过程能对用户透明那就太棒了。事实上，可以通过计算容器 rootfs 挂载点来达到这种目的。

和容器 rootfs 挂载点密不可分的一个概念是 [storage driver](https://docs.docker.com/storage/storagedriver/)。实际使用过程中，用户往往会根据 linux 版本、文件系统类型、容器读写情况等因素选择合适的 storage driver。不同 storage driver 下，容器的 rootfs 挂载点遵循一定规律，因此我们可以根据 storage driver 的类型推断出容器的 rootfs 挂载点，进而采集容器内部日志。下表展示了部分 storage dirver 的 rootfs 挂载点及其计算方法。

| Storage driver | rootfs 挂载点                                | 计算方法                                                     |
| :------------- | :------------------------------------------- | :----------------------------------------------------------- |
| aufs           | /var/lib/docker/aufs/mnt/<id>                | id 可以从如下文件读到。 `/var/lib/docker/image/aufs/layerdb/mounts/<container-id>/mount-id` |
| overlay        | /var/lib/docker/overlay/<id>/merged          | 完整路径可以通过如下命令得到。 `docker inspect -f '{{.GraphDriver.Data.MergedDir}}' <container-id>` |
| overlay2       | /var/lib/docker/overlay2/<id>/merged         | 完整路径可以通过如下命令得到。 `docker inspect -f '{{.GraphDriver.Data.MergedDir}}' <container-id>` |
| devicemapper   | /var/lib/docker/devicemapper/mnt/<id>/rootfs | id 可以通过如下命令得到。 `docker inspect -f '{{.GraphDriver.Data.DeviceName}}' <container-id>` |

示例：

```bash
# 创建容器 tomcat-test
[root@fy-local-2 /]# docker  run -d  --name  tomcat-test  -P  tomcat
36510dd653ae7dcac1d017174b1c38b3f9a226f9c4e329d0ff656cfe041939ff  
# 查看tomcat-test 容器的 挂载点位置
[root@fy-local-2 /]# docker inspect -f '{{.GraphDriver.Data.MergedDir}}' 36510dd653ae7dcac1d017174b1c38b3f9a226f9c4e329d0ff656cfe041939ff  
/var/lib/docker/overlay2/c10ec54bab8f3fccd2c5f1a305df6f3b1e53068776363ab0c104d253216b799d/merged
# 查看挂载点的目录结构
[root@fy-local-2 /]# ls -l /var/lib/docker/overlay2/c10ec54bab8f3fccd2c5f1a305df6f3b1e53068776363ab0c104d253216b799d/merged
total 4
drwxr-xr-x 1 root root  179 May  8 13:05 bin
drwxr-xr-x 2 root root    6 Mar 28 17:12 boot
drwxr-xr-x 1 root root   43 May 22 17:27 dev
lrwxrwxrwx 1 root root   33 May  8 13:08 docker-java-home -> /usr/lib/jvm/java-8-openjdk-amd64
drwxr-xr-x 1 root root   66 May 22 17:27 etc
drwxr-xr-x 2 root root    6 Mar 28 17:12 home
drwxr-xr-x 1 root root    6 May 16 08:50 lib
drwxr-xr-x 2 root root   34 May  6 08:00 lib64
drwxr-xr-x 2 root root    6 May  6 08:00 media
drwxr-xr-x 2 root root    6 May  6 08:00 mnt
drwxr-xr-x 2 root root    6 May  6 08:00 opt
drwxr-xr-x 2 root root    6 Mar 28 17:12 proc
drwx------ 1 root root   27 May 22 17:29 root
drwxr-xr-x 3 root root   30 May  6 08:00 run
drwxr-xr-x 2 root root 4096 May  6 08:00 sbin
drwxr-xr-x 2 root root    6 May  6 08:00 srv
drwxr-xr-x 2 root root    6 Mar 28 17:12 sys
drwxrwxrwt 1 root root   29 May 16 08:50 tmp
drwxr-xr-x 1 root root   19 May  6 08:00 usr
drwxr-xr-x 1 root root   41 May  6 08:00 var
# 查看日志
[root@fy-local-2 /]# ls -l /var/lib/docker/overlay2/c10ec54bab8f3fccd2c5f1a305df6f3b1e53068776363ab0c104d253216b799d/merged/usr/local/tomcat/logs/
total 20
-rw-r----- 1 root root 14514 May 22 17:40 catalina.2019-05-22.log
-rw-r----- 1 root root     0 May 22 17:27 host-manager.2019-05-22.log
-rw-r----- 1 root root  1194 May 22 17:40 localhost.2019-05-22.log
-rw-r----- 1 root root     0 May 22 17:27 localhost_access_log.2019-05-22.txt
-rw-r----- 1 root root     0 May 22 17:27 manager.2019-05-22.log

```



#### 方案四  在代码层中实现直接将日志写入redis

docker  ——》redis ——》Logstash——》Elasticsearch 

通过代码层面，直接将日志写入`redis`,最后写入 `Elasticsearch`。



以上就是对 Docker 日志的所有的概念解释和方提供，具体采用什么方案，根据公司的具体的业务来选择。合适的才是最好的。

