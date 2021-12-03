# Linux运维常见命令及正则

###  常见命令安装

```shell
# netstat   网络
[root@hualaotou ~]# yum install net-tools

# mpstat     CPU监控
[root@hualaotou ~]# yum -y install sysstat

# lsof  进程
[root@hualaotou ~]# yum -y install lsof

# tcpdump  抓包
[root@hualaotou ~]# yum  -y install tcpdump


```



###  命令总结

#####  1、删除0字节文件

```shell
 find -type f -size 0 -exec rm -rf {} \;
```

#####  2、查看进程

​	按照内存从大到小排列

```shell 
ps -e -o "%C : %p : %z : %a"|sort -k5 -nr
```

##### 3、按CPU利用率从大到小排序

```shell
ps -e -o "%C : %p : %z : %a"|sort -nr

# 查看物理CPU个数
cat /proc/cpuinfo| grep “physical id”| sort | uniq| wc -l

# 查看每个物理CPU中的core的个数（即核数）
cat /proc/cpuinfo| grep “cpu cores” | uniq

# 查看逻辑CPU个数
cat /proc/cpuinfo| grep “processor”| wc -l
```

#####  4、打印cache里的URL

```shell
grep -r -a jpg /data/cache/* | strings | grep "http:" | awk -F'http:' '{print "http:"$2;}'
```

##### 5、查看http的并发请求书及其TCP连接状态

```shell
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
```

##### 6、sed替换

```shell
# sed 在这个文里 Root 的一行，匹配 Root 一行，将 no 替换成 yes
sed -i '/Root/s/no/yes/' /etc/ssh/sshd_config
```

##### 7、杀掉MySQL进程

```shell
ps aux |grep mysql |grep -v grep  |awk '{print $2}' |xargs kill -9 (从中了解到awk的用途)

killall -TERM mysqld

kill -9 `cat /usr/local/apache2/logs/httpd.pid`   试试查杀进程PID
```

##### 8、显示运行3级别开启的服务

```shell
ls /etc/rc3.d/S* |cut -c 15-   (从中了解到cut的用途，截取数据)
```

##### 9、如何在编写shell显示多个信息，用EOF

```shell
cat << EOF
+--------------------------------------------------------------+
|       === Welcome to Tunoff services ===                |
+--------------------------------------------------------------+
EOF
```

##### 10、for的巧用（给MySQL创建软连接）

```shell
cd /usr/local/mysql/bin for i in *do ln /usr/local/mysql/bin/$i /usr/bin/$idone
```

##### 11、获取ip地址

```shell
ifconfig eth0 |grep "inet addr:" |awk '{print $2}'| cut -c 6-  或者ifconfig | grep 'inet addr:'| grep -v '127.0.0.1' | cut -d: -f2 | awk '{ print $1}'

[root@hualaotou-es1 ~]# ip a | grep inet | grep -v inet6|grep ens32 |awk '{print $2}' | awk -F "/" '{print $1}'
10.4.7.44
```

##### 12、获取内存的大小

```shell
free -m |grep "Mem" | awk '{print $2}'
```

##### 13、CPU负载

```shell
[root@hualaotou ~]# cat /proc/loadavg
0.00 0.01 0.05 1/142 12989
# 检查前三个输出值是否超过了系统逻辑 CPU 的4倍。

[root@hualaotou ~]# mpstat 1 1


# CPU利用率达到100%怎么排查问题
1、通过top找到CPU占用率高的进程
2、通过top -Hp pid命令查看CPU占比靠前的线程ID
3、再把线程ID转化为16进制，printf "0x%x\n" 74317,得到0x1224d
4、通过命令jstack 72700 | grep ‘0x1224d’ -C5 --color找到有问题的代码
```

##### 14、磁盘空间

```shell
[root@hualaotou ~]# du -cks * | sort -rn | head -n 10

[root@hualaotou ~]# df -Th
```

##### 15、磁盘I/O负载

```shell
[root@hualaotou ~]# iostat -x 1 2
检查I/O使用率（%util）是否超过 100%
```

##### 16、网络负载

```shell
# 网络负载
[root@hualaotou ~]# sar -n DEV
检查网络流量（rxbyt/s, txbyt/s）是否过高

# 网络错误
[root@hualaotou ~]# netstat -i
检查是否有网络错误（drop fifo colls carrier），也可以用命令：# cat /proc/net/dev

# 网络连接数目
[root@hualaotou ~]# netstat -an | grep -E "^(tcp)" | cut -c 68- | sort | uniq -c | sort -n
```

##### 17、进程

```shell
# 进程总数
[root@hualaotou ~]# ps aux | wc -l

# 可运行进程数据
[root@hualaotou ~]# vmstat 1 5

# 进程
[root@hualaotou ~]# top -id 1
观察是否有异常进程出现。

# 杀掉80端口相关的进程
[root@hualaotou ~]# lsof -i :80|grep -v ID|awk '{print "kill -9",$2}'|sh

# 清除僵死进程
[root@hualaotou ~]# ps -eal | awk '{ if ($2 == "Z") {print $4}}' | kill -9

```

##### 18、用户

```shell
[root@hualaotou ~]# who | wc -l
# 检查登录用户是否过多 (比如超过50个)   也可以用命令：# uptime。
```

##### 19、日志

```shell
# 系统日志
[root@hualaotou ~]# grep -i error /var/log/messages
[root@hualaotou ~]# grep -i fail /var/log/messages

# 核心日志
dmesg

 # logwatch –print
配置 /etc/log.d/logwatch.conf，将 Mailto 设置为自己的 email 地址，启动 mail 服务(sendmail或者postfix)，这样就可以每天收到日志报告了。

缺省 logwatch 只报告昨天的日志，可以用  logwatch –print –range all 获得所有的日志分析结果。

可以用 logwatch –print –detail high 获得更具体的日志分析结果(而不仅仅是出错日志)。
```

#####  20、tcpdump抓包

```shell
# tcpdump 抓包，用来防止80端口被人攻击时可以分析数据
[root@hualaotou ~]# tcpdump -c 10000 -i ens32 -n dst port 80 > /root/pkts
```

##### 21、查看系统自启动的服务

```shell
[root@hualaotou ~]# chkconfig --list | awk '{if ($5=="3:on") print $1}'

[root@hualaotou ~]# systemctl list-unit-files

```

##### 22、查看网卡型号

```shell
[root@hualaotou ~]# kudzu --probe --class=network
```

###  常用正则表达式

##### 1、匹配中文字符的正则表达式：

​	**`[\u4e00-\u9fa5]`**

​	 评注：匹配中文还真是个头疼的事，有了这个表达式就好办了 

##### 2、匹配双字节符（包括汉字在内）：

​	**`[^\x00-\xff]`**

​	 评注：可以用来计算字符串的长度（一个双字节字符长度计2，ASCII字符计1） 

##### 3、 匹配空白行的正则表达式： 

​	**` \n\s*\r `**

​	 评注：可以用来删除空白行 

##### 4、 匹配 HTML 标记的正则表达式： 

​	**` <(\S*?)[^>]*>.*?</\1>|<.*? /> `**

​	 评注：网上流传的版本太糟糕，上面这个也仅仅能匹配部分，对于复杂的嵌套标记依旧无能为力 

##### 5、 匹配首尾空白字符的正则表达式： 

​	**` ^\s*|\s*$ `**

​	 评注：可以用来删除行首行尾的空白字符(包括空格、制表符、换页符等等)，非常有用的表达式 

##### 6、 匹配Email地址的正则表达式： 

​	**` \w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)* `**

​	 评注：表单验证时很实用 

##### 7、 匹配网址URL的正则表达式： 

​	**` [a-zA-z]+://[^\s]* `**

​	 评注：网上流传的版本功能很有限，上面这个基本可以满足需求 

##### 8、 匹配帐号是否合法(字母开头，允许5-16字节，允许字母数字下划线)： 

​	**` ^[a-zA-Z][a-zA-Z0-9_]{4,15}$ `**

​	 评注：表单验证时很实用 

##### 9、 匹配国内电话号码： 

​	**` \d{3}-\d{8}|\d{4}-\d{7} `**

​	 评注：匹配形式如 0511-4405222 或 021-87888822 

##### 10、 匹配腾讯QQ号： 

​	**` [1-9][0-9]{4,} `**

​	 评注：腾讯QQ号从10000开始 

##### 11、 匹配中国邮政编码： 

​	**` [1-9]\d{5}(?!\d) `**	

​	 评注：中国邮政编码为6位数字 

##### 12、 匹配身份证号： 

​	**` \d{15}|\d{18} `**

​	 评注：中国的×××为15位或18位 

##### 13、 匹配ip地址： 

​	**` \d+\.\d+\.\d+\.\d+ `**

​	 评注：提取 IP 地址时有用 

##### 14、 匹配特定数字： 

```shell
^[1-9]\d*$   //匹配正整数
^-[1-9]\d*$  //匹配负整数
^-?[1-9]\d*$  //匹配整数
^[1-9]\d*|0$ //匹配非负整数（正整数 + 0）
^-[1-9]\d*|0$  //匹配非正整数（负整数 + 0）
^[1-9]\d*\.\d*|0\.\d*[1-9]\d*$  //匹配正浮点数
^-([1-9]\d*\.\d*|0\.\d*[1-9]\d*)$ //匹配负浮点数
^-?([1-9]\d*\.\d*|0\.\d*[1-9]\d*|0?\.0+|0)$ //匹配浮点数
^[1-9]\d*\.\d*|0\.\d*[1-9]\d*|0?\.0+|0$  //匹配非负浮点数（正浮点数 + 0）
^(-([1-9]\d*\.\d*|0\.\d*[1-9]\d*))|0?\.0+|0$ //匹配非正浮点数（负浮点数 + 0）
```

​	 评注： 处理大量数据时有用，具体应用时注意修正 

##### 15、 匹配特定字符串： 

```shell
^[A-Za-z]+$ //匹配由26个英文字母组成的字符串
^[A-Z]+$ //匹配由26个英文字母的大写组成的字符串
^[a-z]+$ //匹配由26个英文字母的小写组成的字符串
^[A-Za-z0-9]+$ //匹配由数字和26个英文字母组成的字符串
^\w+$ //匹配由数字、26个英文字母或者下划线组成的字符串
```

 	评注：最基本也是最常用的一些表达式 