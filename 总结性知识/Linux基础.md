#  Linux基础

* **常见的Linux发行版**：centos、RHEL、Ubuntu、suse、Debian
* **Linux分区方案**：/分区、swap分区、/boot分区
  * swap：交换分区，虚拟内存，当内存耗尽时，把硬盘当做内存用
  * /boot存放启动文件，例如内核kernel
  * Linux至少应该哪一个分区？ 根分区
* **linux目录结构**
  * usr：系统文件
  * boot：系统启动相关文件
  * etc：配置文件
  * lib：库文件
  * tmp：临时文件
  * var：存放变化文件，数据库、日志、邮件
* **Linux常见文件类型**
  * \- 普通文件（文本文件，二进制文件，压缩文件，电影，图片。。。） 
  * d 目录文件（蓝色）
  * b 设备文件（块设备）存储设备硬盘，U盘 /dev/sda, /dev/sda1 
  * c 设备文件（字符设备）打印机，终端 /dev/tty1, /dev/zero 
  * s 套接字文件 
  * p 管道文件 
  * l 链接文件（淡蓝色）

**堡垒机**：

*  堡垒机，即在一个特定的网络环境下，为了保障网络和数据不受来自外部和内部用户的入侵和破坏，而运用各种技 术手段实时收集和监控网络环境中每一个组成部分的系统状态、安全事件、网络活动，以便集中报警、及时处理及审计定责。
* 功能：登录功能、账号管理、身份认证、资源授权、访问控制

###  命令基础语法

####  ls ：查看

```shell
#ls -d /root/Desktop //显示目录本身 
#ls -l //长格式显示(显示文件的详细信息) 
文件类型\权限 硬链接个数 所有者 所属组 大小 修改时间 名字 
#ls -lh //-h human 人性化显示 以单位显示 
#ls -s //查看大小 
#ls -a //all 显示所有文件 （包括隐藏文件） 
#ls -R dir1 //递归显示文件
#ls -S //按文件的Size排序 
#ls -t //按修改时间排序 
#ls -r //逆序排列reverse 
#ls -i //显示文件的inode号（索引号）

扩展 
	隐藏文件:文件名称前面加“.” 
	. 当前目录 
	.. 上一级目录
```

#### cd：切换路径

```shell
#cd //回家 
#cd +路径 
#cd - //切换到上一次去过的目录 
#cd.. //切换到上级目
```

#### touch：创建文件

```shell
# touch file1.txt //无则创建，有则修改时间 
# touch file3 file4
# touch file{1..20} 
# touch file{a..c} 
# touch xingdian{a,b,c}
```

#### mkdir：创建目录

```shell
# mkdir dir1 
# mkdir /home/dir2 /home/dir3 
# mkdir /home/{dir4,dir5} 
# mkdir -v /home/{dir6,dir7} 
# mkdir -v /hoem/dir8/111/22 
# mkdir -pv /hoem/dir8/111/222 //包括其父母的创建，不会有任何消息输出 
# mkdir -pv /home/{yang/{dir1,111},xingdian}
```

#### cp：复制

```shell
# cp -v anaconda-ks.cfg /home/dir1 //目录 
# cp -v anaconda-ks.cfg /home/dir1/yang.txt //文件 
# cp -rv /etc /home/dir1
# cp -v anaconda-ks.cfg /home/dir90 //没有/home/dir90 
# cp -v anaconda-ks.cfg /home/dir2 
# cp -v file1 !$ 上次执行过的命令的最后一个参数 
# cp -rv /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/passwd /etc/hostname /home/dir2 //将多个文件拷贝到同一个目录 
# cp -r /etc /tmp 
# cp -rf /etc /tmp
```

#### mv：移动

```shell
# mv file1 /home/dir3 将file2移动到/home/dir3 
# mv file2 /home/dir3/file20 将file2移动到/home/dir3，并改名为file20 
# mv file4 file5 将file4重命名为file5，当前位置的移动就是重命名
```

#### rm：删除

```shell
# rm -rf dir1
-r 递归 
-f force强制 
-v 详细过程 
脚本删除: /home/dir1 
rm -rf /home/dir1
```

#### 查看文件内容

```shell
cat 
	-n 显示行号 
	-A 包括控制字符（换行符/制表符）
less more head tail tailf
```

#### grep ：过滤

```shell
# grep 'root' /etc/passwd 
# grep '^root' /etc/passwd 
# grep 'bash$' /etc/passwd 
# grep 'failure' /var/log/secure
```

#### vim ：文件编辑

```shell
命令模式： 
	a. 光标定位 
		hjkl 左 下 上 右 
		home end 0 $ 
		gg G 
		3G 进入第三行 
	b. 文本编辑（少量） 
		y 复制 yy 3yy ygg yG (以行为单位) 
		d 删除 dd 3dd dgg dG (以行为单位) 
		p 粘贴 
		x 删除光标所在的字符 
		D 从光标处删除到行尾 
		u undo撤销 
		r 可以用来修改一个字符 
	c. 进入其它模式 
		a 进入插入模式 
		i 进入插入模式 
		o 进入插入模式 
		A 进入插入模式 
		: 进入末行模式（扩展命令模式） 
		v 进入可视模式 
		ctrl+v 进入可视块模式 
		V 进入可视行模式
		R 进入替换模式 
	可视块模式 ：
		块插入（在指定块前加入字符）： 选择块，I 在块前插入字符， ESC 
		块替换： 选择块，r 输入替换的字符 
		块删除： 选择块，d 
		块复制： 选择块，y 
	扩展命令模式： 
		a. 保存退出 
			:10 进入第10行 
			:w 保存 :q 退出 
			:wq 保存并退出 
			:w! 强制保存 
			:q! 不保存并退出 
			:wq! 强制保存退出 
			:x 保存并退出 ZZ 
		b. 查找替换 
			:范围 s/old/new/选项 
			:1,5 s/root/yang/ 从1－5行的root 替换为yang 
			:5,$ s/root/yang/ $表示最后一行 
			:1,$ s/root/yang/g = :% s/root/yang/g %表示全文 g表示全局 
			:% s#/dev/sda#/var/ccc#g 
			:,8 s/root/yang/ 从当前行到第8行 
			:4,9 s/^#// 4-9行的开头#替换为空 
			:5,10 s/.*/#&/ 5-10前加入#字符 （.*整行 &引用查找的内容） 
		c. 读入文件/写文件(另存为) 
			:w 存储到当前文件 
			:w /tmp/aaa.txt 另存为/tmp/aaa.txt 
			:1,3 w /tmp/2.txt 
			:r /etc/hosts 读入文件到当前行后 
			:5 r /etc/hosts 读入文件到第5行后 
		d. 设置环境 临时设置： 
			:set nu 设置行号 
			:set ic 不区分大小写 
			:set nonu 取消设置行号 
			:set noic 
		永久的环境：修改vim环境配置文件 
			/etc/vimrc 影响所有系统用户 
			~/.vimrc 影响某一个用户 
			# vim ~/.vimrc 
				set ic 
				set nu
```

### 用户管理

* 查看当前登录的用户信息

```shell
查看当前登录的用户信息： 
[root@xingdian ~]# id
uid=0(root) gid=0(root) groups=0(root)
[root@xingdian ~]# id user01 
uid=507(user01) gid=512(user01) groups=512(user01)

系统约定： 
uid: 0 特权用户 
uid: 1~499 系统用户 
uid: 500+ 普通用户
```

#### 组管理

```shell
# 创建组
[root@xingdian ~]# groupadd hr
[root@xingdian ~]# groupadd net01 -g 2000 //添加组net01，并指定gid 2000 
[root@xingdian ~]# grep 'net01' /etc/group //查看/etc/group中组net01信息 
# 删除组
[root@xingdian ~]# groupdel net01 //删除组net01
```

#### 用户管理

```shell
# 创建用户
[root@xingdian ~]# useradd user02 -u 503 //创建用户usr02，指定uid 
[root@xingdian ~]# useradd user03 -d /aaa //创建用户user03 指定家目录 
[root@xingdian ~]# useradd user05 -s /sbin/nologin //创建用户并指定shell 
[root@xingdian ~]# useradd user07 -G hr,it,fd //创建用户，指定附加组 
[root@xingdian~]# useradd user10 -u 4000 -s /sbin/nologin
# 删除用户
[root@xingdian ~]# userdel user10 //删除用户user10，但不删除用户家目录和mail spool
# 更改用户密码
[root@xingdian ~]# passwd alice //root可以给任何用户设置密码

# 组成员管理
[root@xingdian ~]# usermod -G hr niuniu2 //覆盖原有的附加组 
[root@xingdian ~]# usermod -G fd,it niuniu2 
[root@xingdian ~]# usermod -aG hr niuniu2 //增加新的附加组 
[root@xingdian~]# gpasswd -a jack wheel //usermod -aG hr zhuzhu 
[root@xingdian~]# gpasswd -M zhuzhu,maomao100 hr //同时设置多个用户属于某个附加组 
[root@xingdian~]# gpasswd -d zhuzhu hr //删除附加组
[root@xingdian ~]# usermod -s /sbin/nologin niuniu2 //设置新的登录shell
```

#### 用户提权

```shell
1. Switching users with su 
[alice@xingdian ~]$ useradd u1 
-bash: /usr/sbin/useradd: 权限不够 
[alice@xingdian ~]$ su - root password： 
[root@xingdian~]# useradd u1 2. Running commands as root with sudo 
[root@xingdian ~]# useradd yangyang -G wheel 
[root@xingdian ~]# id yangyang
uid=504(yangyang) gid=504(yangyang) 组=504(yangyang),10(wheel)
```

###  权限管理

#### 基本权限

```shell
权限对象：
属主： u   属组： g   其他人: o
基本权限类型：
读：r 4     写：w 2    执行: x 1

```

##### 更改属主属组

```shell
=chown： 
[root@xingdian ~]# chown alice.hr file1 //改属主、属组 
[root@xingdian ~]# chown alice file1 //只改属主 
[root@xingdian ~]# chown .hr file1 //只改属组 
[root@xingdian ~]# chown -R zhouchen.hr dir1 

=chgrp： 
[root@xingdian ~]# chgrp it file1 //改文件属组 
[root@xingdian ~]# chgrp -R it dir1 //改文件属组

2. 更改权限 
=a. 使用符号 
          对象	 赋值符	 权限类型 
			u 		+ 			r 
chmod 		g 		- 			w 		file1 
			o 		= 			x 
			a 
[root@xingdian ~]# chmod u+x file1 //属主增加执行 
[root@xingdian ~]# chmod a=rwx file1 //所有人等于读写执行 
[root@xingdian ~]# chmod a=- file1 //所有人没有权限 
[root@xingdian ~]# chmod ug=rw,o=r file1 //属主属组等于读写，其他人只读 
[root@xingdian ~]# ll file1 //以长模式方式查看文件权限 
-rw-rw-r-- 1 alice it 17 10-25 16:45 file1 //显示的结果

=b. 使用数字 
[root@xingdian ~]# chmod 644 file1 
[root@xingdian ~]# ll file1 -rw-r--r-- 1 alice it 17 10-25 16:45 file1
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211018135038.png)

#### 高级权限

```shell
权限类型：
suid 4  	sgid 2		sticky 1
suid:	只能给命令添加，添加之后，后面的人执行该命令时就拥有命令所有者的权限
sgid:	新建文件继承目录属组<针对目录>
sticky:	用户只能删除自己的文件
2. 更改权限 
=a. 使用符号 
          对象	 赋值符	 权限类型 
			u 		+ 			S 
chmod 		g 		+ 			S 		file1 
			o 		+			t
=b. 使用数字
chmod 4777 file 
chmod 2770 dir 
chmod 3770 dir
```

#### 权限掩码

```shell
[root@xingdian ~]# umask //查看当前用户的umask权限 
0022
修改shell umask值（临时） 
[root@xingdian ~]# umask 000 
[root@xingdian ~]# mkdir dir900
[root@xingdian ~]# touch file900
[root@xingdian ~]# ll -d dir900 file900 
drwxrwxrwx. 2 root root 4096 3月 11 19:44 dir900 
-rw-rw-rw-. 1 root root 0 3月 11 19:44 file900

umask 用户掩码 
创建的用户权限都是644 
创建的目录权限都是755 
文件的默认权限是666，
目录的默认权限是777 
查询：# umask回车

root账户默认umask：022 
普通账户默认umask：002
```

#### 文件属性

```shell
文件系统属性（隐藏权限）：
查看 lsattr 文件
chattr +a 文件 -->允许往文件里追加内容
chattr +i 文件 -->只能看，其他的都不能

```

###  进程管理

* 程序：二进制文件，静态
* 进程：是程序运行的过程，动态，有生命周期的，可以产生和消亡的
  * 进程是已启动的可执行程序的运行实例，实例即运行可执行程序
* 线程：线程是进程之内独立执行的一个单元。对于操作系统而言，其调度单元是线程；
  * 一个进程至少包括一个线程，通常将该线程称为主线程
  * 一个进程从主线程开始，进而创建一个或多个附加线程，就是所谓的基于多线程的多任务

#### 查看进程

##### 静态查看进程

```shell
=ps
[root@xingdian ~]# ps aux | less 
USER 	PID 	%CPU	%MEM 	VSZ 	RSS 	TTY 	STAT	 START 	TIME 	COMMAND 
root 	1 		0.0 	0.0 	2164 	648 	?		 Ss 	08:47 	0:00 	init [5] 
		a 只能查看所有终端进程 
		u 显示进程拥有者 
		x 显示系统内所有进程 
	USER: 运行进程的用户 
	PID： 进程ID 
	%CPU: CPU占用率 
	%MEM: 内存占用率 
	VSZ： 占用虚拟内存 
	RSS: 占用实际内存 驻留内存 
	TTY： 进程运行的终端
    STAT： 进程状态 man ps (/STATE) 
    	R 运行 
    	S 可中断睡眠 Sleep 
    	D 不可中断睡眠 (usually IO) 
    	T 停止的进程 
    	Z 僵尸进程 
    	X 死掉的进程 
    	【了解 】 
    	Ss s进程的领导者，父进程 
    	S< <优先级较高的进程 
    	SN N优先级较低的进程 
    	R+ +表示是前台的进程组 
    	Sl 以线程的方式运行 
    START: 进程的启动时间 
    TIME： 进程占用CPU的总时间 
    COMMAND： 进程文件，进程名
    
[root@xingdian ~]# ps aux --sort %cpu |less 从小到大 
[root@xingdian ~]# ps aux --sort -%cpu |less 从大到小

[root@xingdian ~]# ps -ef 
UID 	PID 	PPID 	C 	STIME 	TTY 	 TIME 		CMD 
root 	1 		0 		0 	08:47 	? 		00:00:00 init [5] 
		-e显示所有进程 
		-l长格式显示 
		-f完整格式
```

##### 动态查看进程

```shell
=top
[root@xingdian ~]# top -d 1 -p 10126 查看指定进程的动态信息 
[root@xingdian ~]# top -d 1 -u apache 查看指定用户的进程 
[root@xingdian ~]# top -d 1 -b -n 2 > top.txt 将2次top信息写入到文件
第一部分：系统整体统计信息
top - 14:49:33 up  5:52,  2 users,  load average: 0.00, 0.01, 0.05
Tasks: 118 total,   1 running, 117 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.1 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2895296 total,  2218536 free,   211928 used,   464832 buff/cache
KiB Swap:  4194300 total,  4194300 free,        0 used.  2509852 avail Mem 


load average: 0.86, 0.56, 0.78 CPU 1分钟，5分钟，15分钟平均负载
	us 用户空间占用CPU百分比 
	sy 内核空间占用CPU百分比 
	ni 用户进程空间内改变过优先级的进程占用CPU百分比 
	id 空闲CPU百分比 
	wa 等待输入输出的CPU时间百分比 
	hi 硬件中断 
	si 软件中断 
	st: 实时
	
第二部分：进程信息 
	命令
	h|?帮助 
	M 按内存的使用排序
	P 按CPU使用排序 
	N 以PID的大小排序 
	R 对排序进行反转 
	f 自定义显示字段 
	1 显示所有CPU的负载 
	z 彩色 
	W 保存top环境设置 ~/.toprc
	

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211018151116.png)

```shell
=htop
[root@hualaotou-es1 ~]#  yum -y  install epel-release
[root@hualaotou-es1 ~]#  yum -y  install htop

	上左区：显示了CPU、物理内存和交换分区的信息； 
	上右区：显示了任务数量、平均负载和连接运行时间等信息； 
	进程区域：显示出当前系统中的所有进程； 
	操作提示区：显示了当前界面中F1-F10功能键中定义的快捷功能。
F1：显示帮助信息 
F2：配置界面中的显示信息； 
F3：进程搜索； 
F4：进程过滤器； 
F5：显示进程树； 
F6：排序； 
F7：减小nice值； 
F8：增加nice值；
F9：杀掉指定进程；

常用参数： u：显示指定用户的进程
```

##### 查看网络进程

```shell
=netstat  ss
#netstat -auntpl | grep 22 
	过滤22号端口 
		-a 所有的进程 
		-u udp进程 
		-n 显示段口号 5900 vnc 21、20 ftp 
		-t tcp进程 
		-p 显示程序的pid 和 名称 
		-l listening 监听的进程 
ss 推荐使用 查看网络进程 
		-a 所有的进程 
		-u udp进程 
		-n 显示段口号 5900 vnc 21、20 ftp 
		-t tcp进程 
		-p 显示程序的pid 和 名称
		-l listening 监听的进程
```

#### 进程管理

```shell
[root@xingdian ~]# kill -l //列出所有支持的信号 编号 信号名
	1) SIGHUP 重新加载配置 PID不变
	2) SIGINT 键盘中断
	3) SIGQUIT 键盘退出
	9) SIGKILL 强制终止
	15) SIGTERM 终止（正常结束），缺省信号
	18) SIGCONT 继续
	19) SIGSTOP 停止
	20)SIGTSTP 暂停
[root@xingdian ~]# systemctl start sshd 
	start：启动 
	stop：停止 
	status：查看状态 
	restart：重新启动
[root@xiaochen ~]# killall vim //给所有vim进程发送15（终止）信号

[root@localhost ~]# pkill -t pts/3 //只是杀掉终端上的进程 
[root@localhost ~]# pkill -9 -t pts/3 //连终端都一起杀掉了
```

#### 进程防护

```shell
1、查看当前有谁在登陆 
# w 
2、查看谁曾经登陆过 
# last 
3、回顾命令历史 
# histrory 
4、检查哪些进程在消耗cpu 
# top 
5、检查所有的系统进程 
# ps aux 
6、检查进程的网络使用情况 
[root@hualaotou-es1 ~]# yum -y install iftop
# iftop 
7、检查哪些进程在监听网络连接 
# lsof -i 
# netstat -anutp
```

### 管道/重定向

#### 文件描述符

```shell
[root@hualaotou-es1 ~]# ls /proc/$$/fd
0  1  2  255
	0: 标准输入
	1: 标准输出
	2: 标准错误
```

####  重定向

```shell
>覆盖 >>追加
正确输入： 1>  1>> 等价于 >  >>
错误输出： 2>  2>>

# 三通定向
|tee file  覆盖
|tee -a file  追加
[root@xingdian ~]# ip a show eth0 |tee file1 |grep 'inet ' |tee file2 |awk -F"/" '{print $1}' |tee file3 |awk '{print $2}' |tee file4
```

#### 管道

```shell
=sort   对字段排序
	-n:按数值，默认按字符排序
	-r:逆序
	
=uniq   去重
	-c:统计重复次数
=awk	打印指定字段
[root@xingdian ~]# awk -F: '{print $NF}' /etc/passwd |sort |uniq -c
	-F: 指定字段分隔符,默认以空格或者是tab分隔 
	$7 第七个字段 
	$NF表示最后一个字段 
	$(NF-1)表示倒数第二个字段
=cut	截取指定字段
[root@hualaotou-es1 ~]# cut -d ":" -f 3 /etc/passwd
	-d 指定字段分隔符
	-f 指定第几段
```

### 存储管理

#### 基本分区

```shell
分区工具
fdisk   gdisk    parted
= fdisk
[root@xingdian ~]# fdisk /dev/sdb

= gdisk
[root@xingdian ~]# yum -y install gdisk 
[root@xingdian ~]# gdisk /dev/sdb

[root@xingdian ~]# mkfs.xfs -f /dev/sdb1 需要先进行格式化
[root@xingdian ~]# mount /dev/sdb1 /opt/ll  临时挂载磁盘
[root@xingdian ~]# vim /etc/fstab   永久挂载
```

#### 增加交换分区

```shell
# 准备分区
[root@xingdian ~]# fdisk /dev/vdb 
[root@xingdian ~]# partprobe /dev/vdb 
[root@xingdian ~]# ll /dev/vdb* 
brw-rw----. 1 root disk 253, 16 12月 6 10:18 /dev/vdb 
brw-rw----. 1 root disk 253, 17 12月 6 10:18 /dev/vdb1
# 初始化
[root@xingdian ~]# mkswap /dev/vdb1
# 挂载
[root@xingdian ~]# blkid /dev/vdb1 
/dev/vdb1: UUID="ea5b1c77-e540-463c-9644-0d75450f8b4c" TYPE="swap" 
[root@xingdian ~]# vim /etc/fstab 
UUID="ea5b1c77-e540-463c-9644-0d75450f8b4c" swap swap default 0 0 
[root@xingdian ~]# swapon -a (读取/etc/fstab) 
[root@xingdian ~]# swapon -s 
Filename 			  Type 		 	 Size 	  Used 	  Priority 
/dev/vdb1 			partition 		524284 		0 		-1
```

#### 逻辑卷LVM

* LVM：逻辑分区管理 
* PV：物理卷在逻辑卷管理系统最底层，可为整个物理硬盘或实际物理硬盘上的分区。
* VG：卷组建立在物理卷上，一卷组中至少要包括一物理卷，卷组建立后可动态的添加卷到卷组中，一个逻辑卷管理系统工程中可有多个卷组。
* LV：逻辑卷建立在卷组基础上，卷组中未分配空间可用于建立新的逻辑卷，逻辑卷建立后可以动态扩展和缩小空间。
* PE：物理区域是物理卷中可用于分配的最小存储单元，物理区域大小在建立卷组时指定，一旦确定不能更改，同一卷组所有物理卷的物理区域大小需一致，新的pv加入到vg后，pe的大小自动更改为vg中定义的pe大小。
* LE：逻辑卷也被划分为被称为LE，逻辑区域是逻辑卷中可用于分配的最小存储单元，逻辑区域的大小取决于逻辑卷所在卷组中的物理区域的大小。

```shell
# 创建逻辑卷
# 1、准备物理磁盘
[root@xingdian ~]# ll /dev/vd{c,d,e} 
brw-rw----. 1 root disk 253, 32 Jun 6 17:38 /dev/vdc 
brw-rw----. 1 root disk 253, 48 Jun 6 17:38 /dev/vdd 
brw-rw----. 1 root disk 253, 64 Jun 6 17:38 /dev/vde
# 2、PV（创建物理卷）
[root@xingdian ~]# pvcreate /dev/vdd 
Physical volume "/dev/vdd" successfully created
[root@xingdian ~]# pvscan 
	PV /dev/vdd 			lvm2 [2.00 GiB] 
	Total: 1 [2.00 GiB] / in use: 0 [0 ] / in no VG: 1 [2.00 GiB]
[root@xingdian ~]# pvs 
	PV 	VG 	Fmt	 Attr 	PSize 	PFree 
	/dev/vdd lvm2 a-- 	2.00g 	2.00g
# 3、vg (创建卷组) 
[root@xingdian ~]# vgcreate vg1 /dev/vdd 
Volume group "vg1" successfully created 
[root@xingdian ~]# vgs 
	VG 	#PV 	#LV 	#SN 	Attr 	VSize 	VFree 
	vg1  1 		0 		0 		wz--n- 	2.00g 	2.00g 
[root@xingdian ~]# vgscan 
	Reading all physical volumes. This may take a while... 
	Found volume group "vg1" using metadata type lvm2 
[root@xingdian ~]# vgdisplay
# 4、lv（创建逻辑卷）
[root@xingdian ~]# lvcreate -l 10 -n lv1 vg1     # -l ：在原来基础上增加
[root@xingdian ~]# lvcreate -L 200M -n lv2 vg1 		# -L：增加到指定大小
[root@xingdian ~]# lvscan 
	ACTIVE '/dev/vg1/lv1' [640.00 MiB] inherit 
	ACTIVE '/dev/vg1/lv2' [256.00 MiB] inherit
# 5、创建文件系统并挂载
[root@xingdian ~]# mkfs.xfs /dev/vg1/lv1 
[root@xingdian ~]# mkfs.ext4 /dev/vg1/lv2 
[root@xingdian ~]# mkdir /mnt/lv1 /mnt/lv2 
[root@xingdian ~]# vim /etc/fstab 
	/dev/vg1/lv1 /mnt/lv1 xfs defaults 0 0 
	/dev/vg1/lv2 /mnt/lv2 ext4 defaults 0 0 
[root@xingdian ~]# mount -a 
[root@xingdian ~]# df 
Filesystem 	       1K-blocks 	Used 		Available	 	Use% 	Mounted on 
/dev/mapper/vg1-lv1 651948 		32928 		619020  		6% 		/mnt/lv1
/dev/mapper/vg1-lv2 245671 		2062 		226406 			1% 		/mnt/lv2

# 扩容
1. pv 
[root@xingdian ~]# pvcreate /dev/vde 
2. vgextend 
[root@xingdian ~]# vgextend vg1 /dev/vde 
	Volume group "vg1" successfully extended 
[root@xingdian ~]# vgs 
	VG 	#PV	 #LV	 #SN 	Attr 	VSize 	VFree 	
	vg1 2 		2 		0 	wz--n- 	3.99g	 3.76g 

==扩大LV lvextend== 
[root@xingdian ~]# vgs 
	VG 		#PV		 #LV 		#SN 	Attr 	VSize 	VFree 
	vg1 	2 			2 		0 		wz--n- 	1.88g 	1.00g
[root@xingdian ~]# lvextend -L 800M /dev/vg1/lv1 增加到800M 
[root@xingdian ~]# lvextend -l +15 /dev/vg1/lv1 在原有的基础上去增加15
```

#### 文件链接

```shell
# 软连接
[root@xingdian ~]# ln -s /file1 /home/file11
[root@xingdian ~]# ll -i /file1 /home/file11     //保持innode号不变
# 硬链接
[root@xingdian ~]# ln /file2 /file2-h1
[root@xingdian ~]# ln /home/ /mnt 
ln: “/home/”: 不允许将硬链接指向目录
```

#### 软硬链接的区别

* 软连接: ln -s  后面跟绝对路径     硬链接: ln  跟路径
* 硬链接
  * 文件在创建时不会产生新的inode号，
  * 在源文件删除后链接文件依然可以用，
  * 文件类型不会发生改变，
  * 不可以实现跨分区创建 ，
  * 对目录不可用。
* 软链接
  * 文件在创建时会产生新的inode号，
  * 在源文件删除后链接文件不可用，
  * 文件类型会发生改变，
  * 可以实现跨分区创建，
  * 对目录可用。

### 常用命令

#### 查看内存

```shell
= free
[root@hualaotou-es1 ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           2827         243         904          17        1680        2382
Swap:          4095           0        4095
	total:总计物理内存的大小。 
	used:已使用多大。 
	free:可用有多少。 
	Shared:多个进程共享的内存总额。 
	Buffers/cached:磁盘缓存的大小。
	
=/proc/meminfo
[root@hualaotou-es1 ~]# cat /proc/meminfo

=清理buff/cache中的内容
[root@hualaotou-es1 ~]# echo 3 > /proc/sys/vm/drop_caches

== vmstat

```

#### 查看磁盘

```shell
=df		列出文件系统的整体磁盘使用量
[root@hualaotou-es1 ~]# df -Th
文件系统                类型      容量  已用  可用 已用% 挂载点
devtmpfs                devtmpfs  1.4G     0  1.4G    0% /dev
tmpfs                   tmpfs     1.4G     0  1.4G    0% /dev/shm
tmpfs                   tmpfs     1.4G   18M  1.4G    2% /run
tmpfs                   tmpfs     1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        56G  1.7G   54G    4% /
/dev/sda1               xfs       497M  173M  324M   35% /boot
tmpfs                   tmpfs     283M     0  283M    0% /run/user/0
	-a：列出所有的文件系统，包括系统特有的/proc等文件系统 
	-k：以KB的容量显示各文件系统 
	-m：以MB的容量显示各文件系统 
	-h：以人们较易阅读的GB,MB,KB等格式自行显示 
	-H：以M=1000K替代M=1024K的进位方式 
	-T：连同该分区的文件系统名称（例如ext3）也列出
	-i：不用硬盘容量，而以inode的数量来显示

=du  评估文件系统的磁盘使用量
[root@hualaotou-es1 ~]# du -h anaconda-ks.cfg 
4.0K	anaconda-ks.cfg
	-a : 列出所有的文件与目录容量，因为默认仅统计目录下面的文件量而已； 
	-h : 以人们较易读的容量格式（G/M）显示； 
	-s : 列出总量，而不列出每个个别的目录占用了容量； 
	-S : 不包括子目录下的总计，与-s有点差别； 
	-k : 以KB列出容量显示； 
	-m : 以MB列出容量显示。

查找某个目录下的文件磁盘使用量
[root@hualaotou-es1 ~]# du --max-depth=1 /data

=lsblk
[root@hualaotou-es1 ~]# lsblk 
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   60G  0 disk 
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 59.5G  0 part 
  ├─centos-root 253:0    0 55.5G  0 lvm  /
  └─centos-swap 253:1    0    4G  0 lvm  [SWAP]
sr0              11:0    1  4.4G  0 rom  
	NAME : 这是块设备名。 
	MAJ:MIN : 本栏显示主要和次要设备号。
	RM : 本栏显示设备是否可移动设备。注意，在本例中设备sdb和sr0的RM值等于1，这说明他们是可移动设备。 
	SIZE : 本栏列出设备的容量大小信息。例如298.1G表明该设备大小为298.1GB，而1K表明该设备大小为1KB。 
	RO : 该项表明设备是否为只读。在本案例中，所有设备的RO值为0，表明他们不是只读的。 
	TYPE :本栏显示块设备是否是磁盘或磁盘上的一个分区。在本例中，sda和sdb是磁盘，而sr0是只读存储				（rom）。 （LCTT译注，此处sr0的RO项没有标记为1，可能存在一些错误？ 
	MOUNTPOINT : 本栏指出设备挂载的挂载点。
	
[root@xingdian ~]# lsblk -a 列出所有设备
[root@xingdian ~]# lsblk -m 列出设备权限和属主
[root@xingdian ~]# lsblk -b /dev/sda 指定设备查看
```

#### 查看CPU

```shell
= top

= htop

= vmstat
[root@hualaotou-es1 ~]# vmstat 
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 2528084      0 148152    0    0     2    13   12   16  0  0 100  0  0
[root@hualaotou-es1 ~]# vmstat 2 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 2527844      0 148284    0    0     2    13   12   16  0  0 100  0  0
 0  0      0 2525168      0 149592    0    0   636     0  374  391  0  1 89 10  0
 0  0      0 2525152      0 149592    0    0     0     0   31   52  0  0 100  0  0
 	(1)进程procs： 
 		r：在运行队列中等待的进程数 。 
 		b：在等待io的进程数 。 
 	(2)Linux 内存监控内存memoy： 
 		swpd：现时可用的交换内存（单位KB）。 
 		free：空闲的内存（单位KB）。 
 		buff: 缓冲去中的内存数（单位：KB）。 
 		cache：被用来做为高速缓存的内存数（单位：KB）。 
 	(3) Linux 内存监控swap交换页面 
 		si: 从磁盘交换到内存的交换页数量，单位：KB/秒。 
 		so: 从内存交换到磁盘的交换页数量，单位：KB/秒。 
 	(4)Linux 内存监控 io块设备: 
 		bi: 发送到块设备的块数，单位：块/秒。 
 		bo: 从块设备接收到的块数，单位：块/秒。 
 	(5)Linux 内存监控system系统： 
 		in: 每秒的中断数，包括时钟中断。 
 		cs: 每秒的环境（上下文）转换次数。 
 	(6)Linux 内存监控cpu中央处理器： 
 		cs：用户进程使用的时间 。以百分比表示。 
 		sy：系统进程使用的时间。 以百分比表示。 
 		id：中央处理器的空闲时间 。以百分比表示。
		wt：等待IO CPU时间
常见诊断： 
	假如 r 经常大于4 ，且 id 经常小于40，表示中央处理器的负荷很重。 
	假如 bi，bo 长期不等于0，表示物理内存容量太小。 
	b 表示阻塞的进程,这个不多说，进程阻塞，大家懂的。 
	swpd 虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了，如果不是程序内存泄露的原因，那么你该升 级内存了或者把耗内存的任务迁移到其他机器。
	si 每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决 掉。
 	so 每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上。。
    in 每秒CPU的中断次数，包括时间中断 
    sy 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。
    
=/proc/cpuinfo
[root@hualaotou-es1 ~]# cat /proc/cpuinfo


=sar  
查看CPU每隔10min的使用情况
[root@hualaotou-es1 ~]# yum install sysstat
[root@hualaotou-es1 ~]# sar


== mpstat
[root@hualaotou-es1 ~]# mpstat
Linux 3.10.0-1062.el7.x86_64 (hualaotou-es1) 	2021年10月19日 	_x86_64_	(4 CPU)

13时36分20秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest  %gnice   %idle
13时36分20秒  all    0.05    0.00    0.07    0.05    0.00    0.00    0.00    0.00    0.00   99.83
[root@hualaotou-es1 ~]# mpstat -P ALL 5 2
	-P {cpu l ALL} 表示监控哪个CPU， cpu在[0,cpu个数-1]中取值
	
==dstat
[root@hualaotou-es1 ~]# yum -y install dstat
[root@hualaotou-es1 ~]# dstat
You did not select any stats, using -cdngy by default.
----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai hiq siq| read  writ| recv  send|  in   out | int   csw 
  0   0 100   0   0   0|8975B   47k|   0     0 |   0     0 |  45    63 
  0   0 100   0   0   0|   0     0 |  66B  886B|   0     0 |  64   100 
	• sys: 内核进程消耗的CPU时间百分比 
		sys 的值高时，说明系统内核消耗的CPU资源多，这并不是良性的表现，我们应该检查原因。 
	• wai: IO等待消耗的CPU时间百分比
    	wa 的值高时，说明IO等待比较严重，这可能是由于磁盘大量作随机访问造成，也有可能是磁盘的带宽出现瓶颈(块操作)。
	• idl: CPU处在空闲状态时间百分比
	
	直接跟数字，表示几秒收集一次数据，默认为一秒；dstat 5表示5秒更新一次 
		-c,--cpu统计CPU状态，包括 user, system, idle（空闲等待时间百分比）, wait（等待磁盘IO）等； 
		-d, --disk 统计磁盘读写状态 
		-D total,sda 统计指定磁盘或汇总信息 
		-l, --load 统计系统负载情况，包括1分钟、5分钟、15分钟平均值 
		-m, --mem 统计系统物理内存使用情况，包括used, buffers, cache, free 
		-s, --swap 统计swap已使用和剩余量 
		-n, --net 统计网络使用情况，包括接收和发送数据 
		-N eth1,total 统计eth1接口汇总流量 
		-r, --io 统计I/O请求，包括读写请求 
		-p, --proc 统计进程信息，包括runnable、uninterruptible、new 
		-y, --sys 统计系统信息，包括中断、上下文切换 
		-t 显示统计时时间，对分析历史数据非常有用 
		--fs 统计文件打开数和inodes数
```

#### 查看负载

```shell
== top
[root@hualaotou-es1 ~]# top
top - 13:30:18 up 16:26,  2 users,  load average: 0.07, 0.06, 0.05


== htop 


== uptime
[root@hualaotou-es1 ~]# uptime 
 13:29:04 up 16:25,  2 users,  load average: 0.04, 0.05, 0.05
 	1分钟平均负载，5分钟平均负载，15分钟平均负载分别是0.04, 0.05, 0.05

== w
[root@hualaotou-es1 ~]# w
 13:30:02 up 16:26,  2 users,  load average: 0.09, 0.06, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1                      079月21 41days  0.08s  0.08s -bash
root     pts/0    10.4.7.1         09:47    2.00s  1.12s  0.06s w

```

#### 查看io

```shell
== iotop
[root@hualaotou-es1 ~]# iotop
	--version #显示版本号 
	-h, --help #显示帮助信息 
	-o, --only #显示进程或者线程实际上正在做的I/O，而不是全部的，可以随时切换按o 
	-b, --batch #运行在非交互式的模式 
	-n NUM, --iter=NUM #在非交互式模式下，设置显示的次数， 
	-d SEC, --delay=SEC #设置显示的间隔秒数，支持非整数值 
	-p PID, --pid=PID #只显示指定PID的信息 
	-u USER, --user=USER #显示指定的用户的进程的信息 
	-P, --processes #只显示进程，一般为显示所有的线程
    -a, --accumulated #显示从iotop启动后每个线程完成了的IO总数 
    -k, --kilobytes #以千字节显示 
    -t, --time #在每一行前添加一个当前的时间
```

#### 查看网络带宽

```shell
== iftop
[root@hualaotou-es1 ~]# iftop
	第一行：带宽显示 
	中间部分：外部连接列表，即记录了哪些ip正在和本机的网络连接 
	中间部分右边：实时参数分别是该访问ip连接到本机2秒，10秒和40秒的平均流量 
	=>代表发送数据，<= 代表接收数据
	底部三行：表示发送，接收和全部的流量 
	底部三行第二列：为你运行iftop到目前流量 
	底部三行第三列：为高峰值 
	底部三行第四列：为平均值
[root@hualaotou-es1 ~]# iftop -i ens32  指定网卡显示

==nload
[root@hualaotou-es1 ~]# nload
	上半部分是：Incoming也就是进入网卡的流量， 
	下半部分是：Outgoing，也就是从这块网卡出去的流量， 
	每部分都有当前流量（Curr）， 
	平均流量（Avg）， 
	最小流量（Min）， 
	最大流量（Max）， 
	总和流量（Ttl）.
	
==nethogs
[root@hualaotou-es1 ~]# nethogs -d 5   设置5秒钟刷新一次，通过-d来指定刷新频率
[root@hualaotou-es1 ~]# nethogs ens32  监视ens32网络带宽
	m : 修改单位 
	r : 按流量排序 
	s : 按发送流量排序 
	q : 退出命令提示符
	
== mtr  网络诊断工具
[root@hualaotou-es1 ~]# yum -y install mtr
[root@hualaotou-es1 ~]# mtr -r www.baidu.com
Start: Tue Oct 19 15:39:48 2021
HOST: hualaotou-es1               Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- gateway                    0.0%    10    0.2   1.6   0.2   2.5   0.6
  2.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
  	丢包率100%说明在进行路由选路 
  	第一列:显示的是IP地址和本机域名，这点和tracert很像 
  	第二列:是显示的每个对应IP的丢包率 
  	第三列:snt:10 设置每秒发送数据包的数量，默认值是10 可以通过参数 -c来指定。 
  	第四列:显示的最近一次的返回时延 
  	第五列:是平均值 这个应该是发送ping包的平均时延 
  	第六列:是最好或者说时延最短的 
  	第七列:是最差或者说时延最常的 第八列:是标准偏差
  	
==traceroute   路由追踪
[root@hualaotou-es1 ~]# yum -y install traceroute
[root@hualaotou-es1 ~]# traceroute www.baidu.com
traceroute to www.baidu.com (36.152.44.95), 30 hops max, 60 byte packets
 1  gateway (10.4.7.254)  0.113 ms  0.050 ms  0.071 ms
 2  * * *

```



#### 文件查找

##### 按文件名

```shell
[root@xingdian ~]# find /etc -name "ifcfg-eth0" 
[root@xingdian ~]# find /etc -iname "ifcfg-eth0" 		//-i忽略大小写
```

##### 按文件大小

```shell
[root@xingdian ~]# find /etc -size +5M 		//大于5M 
[root@xingdian ~]# find /etc -size 5M 
[root@xingdian ~]# find /etc -size -5M 
[root@xingdian ~]# find /etc -size +5M -ls 		//-ls找到的处理动作
```

##### 按时间

```shell
atime,mtime,ctime
[root@xingdian ~]# find /etc -mtime +5		 //修改时间超过5天 
[root@xingdian ~]# find /etc -mtime 5		 //修改时间等于5天 
[root@xingdian ~]# find /etc -mtime -5		 //修改时间5天以内
```

##### 按文件属主、属组

```shell
[root@xingdian ~]# find /home -user jack 		//属主是jack的文件 
[root@xingdian ~]# find /home -group hr 		//属组是hr组的文件 
[root@xingdian ~]# find /home -user jack -group hr 
[root@xingdian ~]# find /home -user jack -a -group hr 
[root@xingdian ~]# find /home -user jack -o -group hr 
	-o 是或者的意思 
	-a 是而且的意思 
	-not 是相反的意思
```

##### 按文件类型

```shell
[root@xingdian ~]# find /dev -type f		 //f普通 
[root@xingdian ~]# find /dev -type d		 //d目录 
[root@xingdian ~]# find /dev -type l		 //l链接 
[root@xingdian ~]# find /dev -type b 		//b块设备 
[root@xingdian ~]# find /dev -type c		 //c字符设备 
[root@xingdian ~]# find /dev -type s		 //s套接字 
[root@xingdian ~]# find /dev -type p		 //p管道文件
```

##### 按文件权限

```shell
[root@xingdian ~]# find . -perm 644 -ls 
root@xingdian ~]# find . -perm -644 -ls 
[root@xingdian ~]# find . -perm -600 -ls 
[root@xingdian ~]# find . -perm -222 -ls
```

##### 按正则表达式

```shell
-regex pattern 
[root@xingdian ~]# find /etc -regex '.*ifcfg-eth[0-9]'
	.* 任意多个字符 
	[0-9] 任意一个数字
```

#### 查找后的动作

```shell
	-print: 显示 
	-ls：类似ls -l的形式显示每一个文件的详细 
	-delete: 删除匹配到的行 
	-ok COMMAND {} \; 每一次操作都需要用户确认,{}表示引用找到的文件,是占位符 
	-exec COMMAND {} \; 每次操作无需确认
[root@xingdian ~]# find /etc -name "ifcfg*" -print 
[root@xingdian ~]# find /etc -name "ifcfg*" -ls 
[root@xingdian ~]# find /etc -name "ifcfg*" -exec cp -rvf {} /tmp \; 
[root@xingdian ~]# find /etc -name "ifcfg*" -ok cp -rvf {} /tmp \; 
root@xingdian ~]# find /etc -name "ifcfg*" -exec rm -rf {} \; 
[root@xingdian ~]# find /etc -name "ifcfg*" -delete

=find结合xargs
[root@xingdian ~]# find . -name "xingdian*.txt" |xargs rm -rf 
[root@xingdian ~]# find /etc -name "ifcfg-eth0" |xargs -I {} cp -rf {} /var/tmp 
[root@xingdian ~]# find . -type f -name "*.txt" |xargs -i cp {} /tmp/
	加 -I 参数 需要事先指定替换字符 
	加 -i 参数直接用 {}就能代替管道之前的标准输出的内容
```

#### 文件打包及压缩

```shell
===打包=== 
[root@xingdian ~]# tar -czf etc1.tar.gz /etc //-z 调用gzip 
[root@xingdian ~]# tar -cjf etc2.tar.bz2 /etc //-j 调用bzip2 
[root@xingdian ~]# tar -cJf etc3.tar.xz /etc //-J 调用xz

===解压，解包=== 
[root@xingdian ~]# tar -tf sys.tar.xz 
[root@xingdian ~]# tar -xzvf etc1.tar.gz 
[root@xingdian ~]# tar -xvf etc1.tar.gz //无需指定解压工具，tar会自动判断 
[root@xingdian ~]# tar -xvf etc2.tar.bz2 -C /tmp //-C重定向到//tmp目录

	-c | --create 建立新的存档 
	-f | --file [HOSTNAME:]F 指定存档或设备

==解压zip ==
[root@xingdian ~]# unzip xxx.zip
```

### 软件安装与卸载

```shell
== yum
[root@xingdian ~]# yum clean all 		// 清除原来旧的YUM 数据库信息 
[root@xingdian ~]# yum makecache 		// 更新新的YUM仓库信息
[root@xingdian ~]#yum repolist all | grep mysql  		//查看所有关于mysql的库 
[root@xingdian ~]#yum -config-manager --enable mysql-community  	//将禁用的yum源库启用
[root@xingdian ~]# yum list httpd 		 //列出资源库中所有可以安装或更新的rpm包
[root@xingdian ~]# yum info httpd 		 //列出资源库中所有可以安装或更新的rpm包的信息
[root@xingdian ~]# yum grouplist  		//安装了的组和可以安装的组一览显示
[root@xingdian ~]# yum -y remove mysql-server 		  //卸载
[root@xingdian ~]# yum history   		 //yum执行的历史记录
[root@xingdian ~]# yum search chinese 		//关注软件包的名 或 描述
[root@xingdian ~]# yum provides pip 		 //查找pip命令属于哪个包提供

==rpm
[root@xingdian ~]#rpm -ivh 软件包名称 
	-i install 
	-vh verbose human 人性化显示
	-q query 查询 
	-l list
	-a all
	-f file
[root@xingdian ~]#rpm  -qf 		 查询某一个文件是哪个软件产生的:
[root@xingdian ~]#rpm -qi 		软件名称  //查询软件详细信息
[root@xingdian ~]#rpm -qc   //查看配置文件
[root@xingdian ~]#rpm -qa   //查询软件全称

[root@xingdian ~]#rpm -e 软件名称     //卸载
	-e erase 
	--force 在安装的时候用(强制覆盖安装) 
	--nodeps 在卸载的时候用(卸载的时候不检查依赖关系)
```

#### 编译安装

* 安装编译依赖环境  yum
* 下载软件  wget
* 解压并切到该解压目录     tar     cd
* 配置  ./configure 
* 编译  make
* 安装  make install

### 计划任务

####  一次性计划任务

```shell
== at
[root@hualaotou-es1 ~]# yum -y install at
[root@hualaotou-es1 ~]# systemctl start atd
[root@hualaotou-es1 ~]# systemctl enable  atd

# 创建计划任务
[root@hualaotou-es1 ~]# at 11:00
at> touch jiajia<EOT>
job 1 at Wed Oct 20 11:00:00 2021
ctrl +d ---->正常结束

查看at计划任务个数：at -l 
[root@hualaotou-es1 ~]# at -l
1	Wed Oct 20 11:00:00 2021 a root
查看详细的计划任务：cat /var/spool/at/a000010180da9e 
[root@hualaotou-es1 ~]# cat /var/spool/at/a00001019fb9b4

删掉计划任务： atrm 3---->工作号 
at -r/-d 3

扩展： 
	midnight:午夜 
	noon：中午 
	teatime ：下午茶 
	23:59 12/31/2018 任务在2018年12月31号23点59分
```

#### 循环计划任务

```shell
[root@hualaotou-es1 ~]#  systemctl status crond.service
# 存储位置
[root@hualaotou-es1 ~]# ls /var/spool/cron/
# 管理方法
crontab -l List the jobs for the current user. 
crontab -r Remove all jobs for the current users. 
crontab -e Edit jobs for the current user. 
管理员可以使用 -u username, 去管理其他用户的计划任务
[root@hualaotou-es1 ~]# crontab -e
# *		 * 		* 		*	 	* 		command   
# 分		时	   日	  月		 周		   命令
示例: 
	00 02 * * * ls 		//每天2:00整 
	00 02 1 * * ls		 //每月1号2:00整 
	00 02 14 2 * ls 		//每年2月14号2:00整 
	00 02 * * 7 ls		 //每周日2:00整 
	00 02 * 6 5 ls		 //每年6月的周五2:00整 
	00 02 14 * 7 ls		 //每月14号2:00整 或者 每周日2:00整，这两个时间都执行 
	00 02 14 2 7 ls 		//每年2月14号2:00整 或者 每周日2:00整，这两个时间都执行 
	00 02 * * * ls 		//每天2:00整 
	* 02 * * * ls 		//每天2:00中的每一分钟 
	* * * * * ls 		//每分钟执行ls 
	* * 14 2 * ls		 //2月14号的每分钟 1440分钟
    */5 * * * * ls		 //每隔5分钟
    00 02 1,5,8 * * ls 		//每月1,5,8号的2:00整 
    00 02 1-8 * * ls 		//每月1到8号的2:00整
```

### 日志

```shell
# 常见日志文件
# tail /var/log/messages 		//系统主日志文件
# tail -f /var/log/secure 		//认证、安全 
# tail /var/log/cron 		//crond、at进程产生的日志 
# tail /var/log/yum.log 		//yum

# w //当前登录的用户即: /var/log/wtmp日志 
# last //最近登录的用户 /var/log/btmp 
# lastlog //所有用户的登录情况 /var/log/lastlog
```

#### 集中式日志管理

```shell
自定义日志 
自己定义日志的名字和位置 
查看日志文件是否起来：systemctl status rsyslog
#vim /etc/rsyslog.conf -----> 日志的主配置文件 include 包含
[root@hualaotou-es1 ~]# vim /etc/rsyslog.conf

# 日志级别
	debug：最低的，一般不用 
	info：安装信息，警告信息，错误信息 
	notice：相当与提示 
	warn/warning：警告，错误 
	error/err：错误，严重错误 
	alert：告警，表示已经出现问题 
	emerg：恐慌级别

例：定义sshd日志： 
1.修改sshd服务主配置文件： 
将/etc/ssh/sshd_config 的#SyslogFacility AUTHPRIV改为 SyslogFacility local2 
SyslogFacility local5 //设置ssh的日志定义由local5设备来记录 
2.在rsyslog的主配置文件里加上 local5.info /var/log/ssh 
3.重启服务
```

#### 日志切割

```shell
logrotate日志轮转(切割)
[root@hualaotou-es1 ~]# rpm -qa |grep logrotate
logrotate 配置文件： 
	/etc/logrotate.conf (决定每个日志文件如何轮转) 
	/etc/logrotate.d/*

=====主配置文件
[root@hualaotou-es1 ~]# vim /etc/logrotate.conf 
weekly 		//轮转的周期，一周轮转 
rotate 4 		//保留4份 
create 		//轮转后创建新文件 
dateext 		//使用日期作为后缀 
#compress 		//是否压缩


	/var/log/wtmp { 		//对该日志文件设置轮转的方法 
		monthly 		//一月轮转一次 
		minsize 1M 		//最小达到1M才轮转，即到了规定的时间未达到大小不会轮转 
		create 0664 root utmp 		//轮转后创建新文件，并设置权限属主和属组 
		rotate 1 		//保留一份 
		}
	/var/log/btmp { 
		missingok 		//丢失不提示 
		monthly 		//每月轮转一次 
		create 0600 root utmp 		//轮转后创建新文件，并设置权限 
		rotate 1 		//保留一份 
		}
```

### 时间服务器

```shell
[root@hualaotou-es1 ~]# yum -y install ntp
[root@hualaotou-es1 ~]# systemctl start ntpd
[root@hualaotou-es1 ~]# systemctl enable  ntpd
		ntp  			123/udp
# vim /etc/ntp.conf //配置文件全部删掉，只要下面三行
restrict default nomodify //不允许客户端登录，也不允许客户端修改 
server 127.127.1.0 //使用本地的bios时间，自己跟自己同步 
fudge 127.127.1.0 stratum 10 //定义级别，范围0-16，越小越精准
# systemctl restart ntpd
三、配置NTP客户端 
# ntpdate -b 172.16.110.1 //手动时间同步 -b加速初始化同步 
# crontab -e 
01 * * * * ntpdate 172.16.110.1
```

###  运行级别

* 0 关机 
* 1 特权模式（但用户模式） 
* 2 不可以使用网络文件系统的多用户模式 几乎不用 
* 3 文本模式 （多用户模式） 
* 4 预留 
* 5 图形模式 （多用户模式） 
* 6 重启 

### 远程拷贝

```shell
==scp
# scp不支持增量拷贝，即使文件存在也不询问，scp是基于ssh的，没有装ssh，scp也就用不了
将自己服务器上面的文件推送到其他的主机上面 
[root@xingdian ~]# scp -r /etc 192.168.5.32:/tmp 
将其他主机上面的文件推送到自己的服务器上面 
[root@xingdian ~]# scp -r 192.168.5.32:/dir1 /root

==rsync
# 用rsync命令的前提是两端都装了rsync，否则用不了 支持增量同步
[root@hualaotou01 ~]# yum -y install rsync
[root@hualaotou01 data]# rsync -vaz /root/data/ 10.4.7.55:/data   //将自己服务器上面的文件推送到其他的主机上面
[root@hualaotou01 data]# rsync -vaz 10.4.7.55:/data /data  //将其他主机上面的文件推送到自己的服务器上面
	注意： 
		-v, --verbose 详细模式输出 
		-q, --quiet 精简输出模式 
		-a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性 
		-z, --compress 对备份的文件在传输时进行压缩处理	
```

### 免密登录

```shell
# hualaotou01远程登录hualaotou02不需要再次输入密码
1、客户端生成公钥和私钥
[root@hualaotou01 ~]# ssh-keygen       //一路回车
[root@hualaotou01 ~]# cd ~/.ssh/
[root@hualaotou01 .ssh]# ll
总用量 12
-rw-------. 1 root root 1679 10月 20 09:56 id_rsa
-rw-r--r--. 1 root root  398 10月 20 09:56 id_rsa.pub
-rw-r--r--. 1 root root  171 10月 20 09:48 known_hosts
2、将公钥传至所需要登录的服务器
[root@hualaotou01 .ssh]# ssh-copy-id -i 10.4.7.55
3、实现免密登录
[root@hualaotou01 .ssh]# ssh 10.4.7.55
Last login: Wed Oct 20 09:40:54 2021 from 10.4.7.1

如果推送公钥之后不能实现使用下面的方法解决： 在Server服务器上加载私钥文件 
[root@hualaotou01 .ssh]# ssh-add id_rsa 
Identity added: id_rsa (id_rsa)

上面方法如果出现错误，使用下面下面方法解决 
[root@hualaotou01 .ssh]# ssh-agent bash 
[root@hualaotou01 .ssh]# ssh-add id_rsa Identity added: id_rsa (id_rsa)


用户免密登录
在ssh目录下创建authorized_keys文件，权限为600
[root@hualaotou01 .ssh]# touch authorized_keys
[root@hualaotou01 .ssh]# chmod 600 authorized_keys

将需要登录的用户公钥添加至该文件即可

```

### FTP服务器

```shell
协议：ftp 文件传输协议 
端口： 
	建立tcp连接： 21 
	传输数据：20 
	1024+的随即端口 
客户端软件： 浏览器 资源管理器
安装软件
	客户端
		lftp-4.0.9-1.el6.x86_64 
		ftp-0.17-53.el6.x86_64 
	服务端
		vsftpd-2.2.2-11.el6.x86_64
		
使用vsftpd: 
	下载上传
	匿名账户 
	本地账户 
	虚拟账户(扩展) 
	限速并发
[root@hualaotou01 ~]# yum -y install vsftpd
[root@hualaotou01 ~]# systemctl start vsftpd
[root@hualaotou01 ~]# systemctl enable vsftpd 
[root@hualaotou01 ~]# systemctl stop firewalld && setenforce 0
浏览器访问：ftp://10.4.7.44/

# 匿名登录
匿名账户默认下载目录（匿名账户根目录）
[root@hualaotou01 ~]# cd /var/ftp/
创建上传目录，权限777
[root@hualaotou01 ftp]# mkdir hualaotou
[root@hualaotou01 ftp]# chmod 777 hualaotou/

# 用户登录
[root@hualaotou01 data]# useradd  -d /jiajia -G ftp jiajia
使用用户登录的时候，共享目录默认为用户的家目录


开启上传
[root@hualaotou01 ~]# vim /etc/vsftpd/vsftpd.conf
27  anon_upload_enable=YES 		//上传文件 
31  anon_mkdir_write_enable=YES 		//上传目录

vsftpd扩展功能 
常用全局配置 
	listen_address=192.168.4.1 //设置监听的IP 地址 和setproctitle_enable=yes 一起用 
	listen_port=21 //设置监听FTP 服务的端口号 
	write_enable=YES //是否启用写入权限 
	download_enable=YES //是否允许下载文件 
	userlist_enable=YES //是否启用user_list 列表文件 
	userlist_deny=YES //是否禁用user_list 中的用户
常用的匿名FTP 配置项 
	anonymous_enable=YES //启用匿名访问 
	anon_umask=022 //匿名用户所上传文件的权限掩码 
	anon_root=/var/ftp //匿名用户根目录 
	anon_upload_enable=YES //允许上传文件 
	anon_mkdir_write_enable=YES //允许创建目录
    anon_other_write_enable=YES //开放其他写入权
常用的本地用户FTP 配置项 
	lcal_enable=YES //是否启用本地系统用户 
	local_umask=022 //本地用户所上传文件的权限掩码 
	local_root=/var/ftp //本地账户ftp根目录
单独对某一个账户设置ftp选项： 
	chroot_local_user=YES //限制所有本地用户在家目录里
另一种指定用户限制,非限制用户依旧可以畅通无阻 
	黑名单： 
	chroot_local_user=NO 
	chroot_list_enable=YES 
	chroot_list_file=/etc/vsftpd/chroot_list 凡是名单中的人不允许离开HOME 目录 
	建立黑名单文件. 
	[root@mail vsftpd]# cat chroot_list 
		wing 
		user1
	白名单：
	chroot_local_user=YES 
	chroot_list_enable=YES 
	chroot_list_file=/etc/vsftpd/chroot_list 凡是名单中的人允许离开HOME 目录
	建立白名单文件. 
	[root@hualaotou01 vsftp]#cat chroot_list 
		wing
        user1 
     只要存在于ftpusers 里的用户都是禁止登陆ftp的
ftp的主动模式和被动模式 
	vsftpd的被动模式是默认开启的，可以关闭，主动模式永远开启，不能关闭 
	主动模式： 21端口负责连接，20端口负责传输数据 
		client      server 
		1024-------->21 负责连接 
		1024<--------20 负责传输 
		高安全区域 防火墙 低安全区域 
	被动模式： 
		client       server 
		1024-------->21 负责会话连接 
		1024-------->1024+存放数据等待客户端来拿数据
		
	使用什么模式连接服务器是客户端说了算 
	能不能接受那个模式是服务器说了算
	服务器关闭被动模式： 
	[root@xingdian ~]# vim /etc/vsftpd/vsftpd.conf 
		pasv_enable=NO
	
```

####  存储共享

```shell
==nfs
1. 软件包： nfs-utils
2. 端口： 2049/tcp
3. 配置文件：/etc/exports

nfs服务端：
[root@hualaotou01 ~]#  yum -y install nfs-utils
[root@hualaotou01 ~]# systemctl start nfs
[root@hualaotou01 ~]# systemctl enable nfs

# 创建共享目录
[root@hualaotou01 ~]# mkdir /love/girl01
[root@hualaotou01 ~]# mkdir /love/girl02

# 配置共享方式
[root@hualaotou01 ~]# vim /etc/exports
/love/girl01            *(ro,sync)     # 以只读方式共享
/love/girl02            10.4.7.0/24(rw,sync,no_root_squash)  # 以读写方式共享

	sync：sync传输过程中将数据直接写入内存和硬盘 
	no_root_squash：当登录NFS主机使用共享目录的使用者是root时，其权限将被转换成为匿名使用者，通常它的UID与 GID都会变成nobody身份。
	
# 刷新共享
[root@hualaotou01 ~]# exportfs -rv
exporting 10.4.7.0/24:/love/girl02
exporting *:/love/girl01

nfs客户端：
# 查看存储端共享
[root@hualaotou02 ~]# showmount -e 10.4.7.44
Export list for 10.4.7.44:
/love/girl01 *
/love/girl02 10.4.7.0/24
# 挂载
[root@hualaotou02 ~]# mkdir -p /data/love0{1,2} 
[root@hualaotou02 ~]# cd /data/
[root@hualaotou02 data]# ll
总用量 176860
drwxr-xr-x. 2 root root        6 10月 20 13:30 love01
drwxr-xr-x. 2 root root        6 10月 20 13:30 love02

[root@hualaotou02 data]# vim /etc/fstab
10.4.7.44:/love/girl01  /data/love01    nfs defaults,_netdev 0 0
10.4.7.44:/love/girl02  /data/love02    nfs defaults,_netdev 0 0

[root@hualaotou02 data]# mount -a

```

