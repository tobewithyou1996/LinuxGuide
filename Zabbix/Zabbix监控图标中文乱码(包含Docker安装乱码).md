最近在看 `Zabbix 4.0` 版本的官方文档，搭建后图表使用中文发现还是有乱码。之前在 3.0 版本的时候也遇到过，之前有记录。现在针对2个版本的乱码问题的解决做下记录。

### Zabbix 4.0  版本

乱码之前的图表中文显示：

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_4.0_%E4%B8%AD%E6%96%87%E4%B9%B1%E7%A0%81.png)



解决办法就是上传中文字体库到  `Zabbix  server`  。替换原来图表使用的字体。

#### 解决思路

先找到图表使用的字体，我们在 ` /usr/share/zabbix/assets/fonts`（yum 安装） 可以看到字体文件 `graphfont.ttf` ，这个文件就是图表使用的字体。(如果在该路径找不到此字体，请检查版本或者使用 Find 查找)。

```bash
[root@localhost fonts]# ls -l /usr/share/zabbix/assets/fonts
total 0
lrwxrwxrwx 1 root root 33 Jun 10 15:17 graphfont.ttf -> /etc/alternatives/zabbix-web-font

```



我们可以看到该字体是链接到 `/etc/alternatives/zabbix-web-font`，我们进行查看 `/etc/alternatives/zabbix-web-font`。发现它链接到了  `/usr/share/fonts/dejavu/DejaVuSans.ttf`

```bash
[root@localhost fonts]# ll -h /etc/alternatives/zabbix-web-font
lrwxrwxrwx 1 root root 38 Jun 13 14:58 /etc/alternatives/zabbix-web-font -> /usr/share/fonts/dejavu/DejaVuSans.ttf
[root@localhost fonts]# ls -l  /usr/share/fonts/dejavu/DejaVuSans.ttf
-rw-r--r-- 1 root root 720012 Feb 27  2011 /usr/share/fonts/dejavu/DejaVuSans.ttf
```

也就是我们的图表使用的字体`graphfont.ttf`  最终是指向  `/usr/share/fonts/dejavu/DejaVuSans.ttf`。

理清楚了这个，我们就可以去找一个中文字体，然后上传到  `/usr/share/fonts/dejavu/`，然后让 `/etc/alternatives/zabbix-web-font`  链接到 `/usr/share/fonts/dejavu/` 里我们上传的新的中文字体。

#### 解决操作

- 找中文字体

  我们直接从我们的windows 系统里面找中文字体。默认路径为 `C:\Windows\Fonts`。我们使用的是楷体。上传到我们 `Zabbix server 服务器` 的 `/usr/share/fonts/dejavu/`

  ![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_4.0%E4%B8%AD%E6%96%87%E5%AD%97%E4%BD%93.png)

  ```
  [root@localhost fonts]# ls -l  /usr/share/fonts/dejavu/
  total 16800
  -rw-r--r-- 1 root root   611212 Feb 27  2011 DejaVuSans-BoldOblique.ttf
  -rw-r--r-- 1 root root   672300 Feb 27  2011 DejaVuSans-Bold.ttf
  -rw-r--r-- 1 root root   580168 Feb 27  2011 DejaVuSansCondensed-BoldOblique.ttf
  -rw-r--r-- 1 root root   631992 Feb 27  2011 DejaVuSansCondensed-Bold.ttf
  -rw-r--r-- 1 root root   576004 Feb 27  2011 DejaVuSansCondensed-Oblique.ttf
  -rw-r--r-- 1 root root   643852 Feb 27  2011 DejaVuSansCondensed.ttf
  -rw-r--r-- 1 root root   345204 Feb 27  2011 DejaVuSans-ExtraLight.ttf
  -rw-r--r-- 1 root root   611556 Feb 27  2011 DejaVuSans-Oblique.ttf
  -rw-r--r-- 1 root root   720012 Feb 27  2011 DejaVuSans.ttf
  -rw-r--r-- 1 root root 11787328 Aug  9  2018 simkai.ttf
  ```

  楷体也就是  `simkai.ttf`

- 替换字体为   `simkai.ttf`

  ```bash
  [root@localhost fonts]# rm -f /etc/alternatives/zabbix-web-font 
  [root@localhost fonts]# ln -s  /usr/share/fonts/dejavu/simkai.ttf   /etc/alternatives/zabbix-web-font
  ```

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_4.0_%E8%A7%A3%E5%86%B3%E4%B9%B1%E7%A0%81.png)

### Zabbix 3.0 版本

 图表乱码

![](https://djxblog.oss-cn-shenzhen.aliyuncs.com/picture/Zabbix/zabbix_3.0_%E5%9B%BE%E8%A1%A8%E4%B8%AD%E6%96%87%E4%B9%B1%E7%A0%81.png)

解决办法和上面大同小异，也是替换字体。

这里说下不同之处。就是 上面的 4.0 版本的 `graphfont.ttf` 字体路径是在 `/usr/share/zabbix/assets/fonts`，而 3.0 版本的字体路径是在 `/usr/share/zabbix/fonts` 。 其他的操作是一致的。





#### Zabbix 4.0  Docker 版本 图表乱码问题解决

字体文件存放于镜像 `zabbix-web-nginx-mysql` 的 `/usr/share/zabbix/assets/fonts/`目录下。

```bash
docker  cp  /tmp/SIMKAI.TTF   c9e36aa249a3:/usr/share/zabbix/assets/fonts/
```

然后我们登录到容器里面 

```bash
 # 将后缀名 TTF 改为 ttf
 [root@c9e36aa249a3 fonts]# mv /usr/share/zabbix/assets/fonts/SIMKAI.TTF  /usr/share/zabbix/assets/fonts/SIMKAI.ttf
 # 编辑文件 /usr/share/zabbix/include/defines.inc.php，大约在69行。将 DejaVuSans  更改为 SIMKAI
[root@c9e36aa249a3 fonts]#  vi /usr/share/zabbix/include/defines.inc.php
# 更改前  
define('ZBX_GRAPH_FONT_NAME',           'DejaVuSans'); // font file name
# 更改后
define('ZBX_GRAPH_FONT_NAME',           'SIMKAI'); // font file name
```

然后刷新界面，就可以正常显示了。

如果是使用的 镜像 `zabbix-web-apache-mysql` ，和 镜像 `zabbix-web-nginx-mysql`  的操作一致。

