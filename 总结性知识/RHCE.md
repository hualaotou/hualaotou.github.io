# 红帽认证

## RHCE

### ssh通过firewalld实现访问控制

**拒绝某个网段访问ssh**

```shell
[root@server0 ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 日 2021-09-26 10:39:58 CST; 19min ago
     Docs: man:firewalld(1)


[root@server0 ~]# firewall-cmd --permanent --add-rich-rule=' rule family="ipv4" source address="172.24.3.0/24" service name="ssh" reject'
success

[root@server0 ~]# firewall-cmd --permanent --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="172.24.3.0/24" service name="ssh" reject

```

### alias  别名设置

```shell
[root@server0 ~]# vim /etc/profile

alias psnew='ps -Ao user,pid,ppid,command'

[root@server0 ~]# source /etc/profile

[root@server0 ~]# psnew 
USER        PID   PPID COMMAND
root          1      0 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
root          2      0 [kthreadd]
root          4      2 [kworker/0:0H]
```

###  samba共享

#####  部署及挂载

```shell
配置server0服务器SMB,工作组为STAFF,共享目录/smb1, 共享名smb1,只有10.4.7.*网段中主机访问
共享smb1,smb1必须可浏览;用户jiajia必须能够读取共享中的内容,密码tianyun。

# 安装软件包
[root@server0 ~]# yum -y install samba samba-client cifs-utils

# 创建共享目录
[root@server0 ~]# mkdir /smb1

# 设置用户的Samba密码
[root@server0 ~]# id jiajia
uid=1000(jiajia) gid=1000(jiajia) 组=1000(jiajia)
[root@server0 ~]# smbpasswd -a jiajia
New SMB password:
Retype new SMB password:
Added user jiajia.

# 修改配置文件
[root@server0 ~]# vim /etc/samba/smb.conf
#在末尾添加
[smb1]
        path = /smb1
        hosts allow = 10.4.7.

# 启动进程
[root@server0 ~]# systemctl  start nmb smb
[root@server0 ~]# systemctl  enable nmb smb
Created symlink from /etc/systemd/system/multi-user.target.wants/nmb.service to /usr/lib/systemd/system/nmb.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/smb.service to /usr/lib/systemd/system/smb.service.

# 防火墙设置
[root@server0 ~]# firewall-cmd --permanent --add-service=samba
success
[root@server0 ~]# firewall-cmd --reload
success

# 设置selinux
[root@server0 ~]# chcon -R -t samba_share_t /smb1
[root@server0 ~]# ll -dZ /smb1
drwxr-xr-x. root root unconfined_u:object_r:samba_share_t:s0 /smb1

# 客户端测试
[root@desktop0 ~]# yum -y install samba-client cifs-utifs

# 客户端临时挂载
[root@desktop0 ~]# mount -t cifs -o user=jiajia,pass=tianyun //server0/smb1 /mnt
[root@desktop0 ~]# df -Th
文件系统                类型      容量  已用  可用 已用% 挂载点
devtmpfs                devtmpfs  1.4G     0  1.4G    0% /dev
tmpfs                   tmpfs     1.4G     0  1.4G    0% /dev/shm
tmpfs                   tmpfs     1.4G  9.3M  1.4G    1% /run
tmpfs                   tmpfs     1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        56G  1.4G   55G    3% /
/dev/sda1               xfs       497M  143M  355M   29% /boot
tmpfs                   tmpfs     283M     0  283M    0% /run/user/0
//server0/smb1          cifs       56G  1.4G   55G    3% /mnt

# 客户端测试完卸载
[root@desktop0 ~]# umount /mnt/
```

##### 多用户操作

```shell
配置server0服务器samba,共享目录/smb2,共享名smb2,只有10.4.7.*网段中主机访问。用户jiajia读取, jiajia2读写,密码都为tianyun;desktop0以multiuser方式自动挂接到/mnt/smb2

# 服务端
[root@server0 ~]# id jiajia
uid=1000(jiajia) gid=1000(jiajia) 组=1000(jiajia)
[root@server0 ~]# id jiajia2
uid=1001(jiajia2) gid=1001(jiajia2) 组=1001(jiajia2)
[root@server0 ~]# smbpasswd  -a jiajia2
New SMB password:
Retype new SMB password:
Added user jiajia2.

[root@server0 ~]# mkdir /smb2
[root@server0 ~]# chcon -R -t samba_share_t /smb2
[root@server0 ~]# ll -dZ /smb2
drwxr-xr-x. root root unconfined_u:object_r:samba_share_t:s0 /smb2
[root@server0 ~]# setfacl -m u:jiajia2:rwx /smb2
[root@server0 ~]# getfacl  /smb2
getfacl: Removing leading '/' from absolute path names
# file: smb2
# owner: root
# group: root
user::rwx
user:jiajia2:rwx
group::r-x
mask::rwx
other::r-x

[root@server0 ~]# vim /etc/samba/smb.conf
[smb2]
        path = /smb2
        hosts allow = 10.4.7.
        valid users = jiajia,jiajia2
        write list = jiajia2

[root@server0 ~]# systemctl restart nmb smb

# 客户端
[root@desktop0 ~]# mkdir  /mnt/smb2
[root@desktop0 ~]# vim /etc/fstab

//server0/smb2  /mnt/smb2       cifs defaults,user=jiajia2,pass=tianyun,multiuser 0 0
# 挂载目录	#挂载地点	#类型		# 用户	# 密码	#多用户挂载 	# 开机不备份、不检测

[root@desktop0 ~]# mount -a
[root@desktop0 ~]# df -Th
文件系统                类型      容量  已用  可用 已用% 挂载点
devtmpfs                devtmpfs  1.4G     0  1.4G    0% /dev
tmpfs                   tmpfs     1.4G     0  1.4G    0% /dev/shm
tmpfs                   tmpfs     1.4G  9.3M  1.4G    1% /run
tmpfs                   tmpfs     1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        56G  1.4G   55G    3% /
/dev/sda1               xfs       497M  143M  355M   29% /boot
tmpfs                   tmpfs     283M     0  283M    0% /run/user/0
//server0/smb2          cifs       56G  1.4G   55G    3% /mnt/smb2

# 多用户权限测试

[root@desktop0 ~]# id jiajia
uid=1000(jiajia) gid=1000(jiajia) 组=1000(jiajia)
[root@desktop0 ~]# id jiajia2
uid=1001(jiajia2) gid=1001(jiajia2) 组=1001(jiajia2)

# 切换用户到jiajia

[root@desktop0 ~]# su jiajia
[jiajia@desktop0 root]$ ls /mnt/smb2 
ls: 无法访问/mnt/smb2: 权限不够
# 需要重新获取身份令牌
[jiajia@desktop0 root]$ cifscreds add server0
Password: # 密码为Samba的密码

[jiajia@desktop0 root]$ ls /mnt/smb2 
a.txt 
[jiajia@desktop0 root]$ cat /mnt/smb2/a.txt 
hello world   # jiajia用户拥有读权限

[jiajia@desktop0 root]$ echo "ni hao " > /mnt/smb2/b.txt
bash: /mnt/smb2/b.txt: 权限不够   # jiajia用户没有写权限

# 切换用户到jiajia2
[jiajia2@desktop0 root]$ cifscreds update server0
Password:  #密码为Samba的密码
[jiajia2@desktop0 root]$ ls /mnt/smb2/
a.txt
[jiajia2@desktop0 root]$ cat /mnt/smb2/a.txt 
hello world   # jiajia2用户拥有读权限
[jiajia2@desktop0 root]$ echo "ni hao" > /mnt/smb2/b.txt
[jiajia2@desktop0 root]$ ls /mnt/smb2/
a.txt  b.txt   #  jiajia2用户拥有写权限


```

### NFS配置

```shell
配置server0 NFS服务
以只读的方式共享目录/nfs1,只能被10.4.7.*网段中主机访问;
以读写的方式共享目录/nfs2,能被10.4.7.*网段中主机访问
访问/nfs2需要Kerberos安全加密,密钥为 http://classroom.example.com/pub/keytabs/server0.keytab
目录/nfs2应包含名为private拥有者为jiajia5的子目录,用户jiajia5能以读写的方式访问/nfs2/private

# 服务端

# 安装必要的软件包
[root@server0 ~]# yum -y install nfs-utils

# 创建必要的目录
[root@server0 ~]# mkdir /nfs1
[root@server0 ~]# mkdir  -p /nfs2/private

[root@server0 ~]# id jiajia5
uid=1002(jiajia5) gid=1002(jiajia5) 组=1002(jiajia5)

[root@server0 ~]# chown jiajia5 /nfs2/private
[root@server0 ~]# ll -d /nfs2/private
drwxr-xr-x. 2 jiajia5 root 6 9月  26 14:56 /nfs2/private

# 配置输出文件
[root@server0 ~]# vim /etc/exports
/nfs1   10.4.7.0/24(ro,sync)

# 启动nfs服务
[root@server0 ~]# systemctl restart nfs-server
[root@server0 ~]# systemctl enable  nfs-server
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.

# 配置防火墙
[root@server0 ~]# firewall-cmd --permanent --add-service=nfs
success
[root@server0 ~]# firewall-cmd --reload
success



# 客户端

[root@desktop0 ~]# yum -y install nfs-utils
[root@desktop0 ~]# mkdir /mnt/nfs1
[root@desktop0 ~]# vim /etc/fstab
server0:/nfs1   /mnt/nfs1       nfs     defaults        0 0

[root@desktop0 ~]# mount -a
[root@desktop0 ~]# df -Th
文件系统                类型      容量  已用  可用 已用% 挂载点
devtmpfs                devtmpfs  1.4G     0  1.4G    0% /dev
tmpfs                   tmpfs     1.4G     0  1.4G    0% /dev/shm
tmpfs                   tmpfs     1.4G  9.3M  1.4G    1% /run
tmpfs                   tmpfs     1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        56G  1.4G   55G    3% /
/dev/sda1               xfs       497M  143M  355M   29% /boot
tmpfs                   tmpfs     283M     0  283M    0% /run/user/0
//server0/smb2          cifs       56G  1.4G   55G    3% /mnt/smb2
server0:/nfs1           nfs4       56G  1.4G   55G    3% /mnt/nfs1
[root@desktop0 ~]# ls /mnt/nfs1/
aa.txt
[root@desktop0 ~]# cat /mnt/nfs1/aa.txt 
hello world
[root@desktop0 ~]# touch /mnt/nfs1/file1
touch: 无法创建"/mnt/nfs1/file1": 只读文件系统

# 可以在服务端查看输出文件
[root@server0 ~]# exportfs -v
/nfs1         	10.4.7.0/24(sync,wdelay,hide,no_subtree_check,sec=sys,ro,secure,root_squash,no_all_squash)


```

##### 加密共享

```shell
# 服务端
# 下载key文件
[root@server0 ~]# wget  http://classroom.example.com/pub/keytabs/server0.keytab -O /etc/krb5.keytab

[root@server0 ~]# vim /etc/exports
/nfs2   10.4.7.0/24(rw,sync,sec=krb5p)

[root@server0 ~]# vim /etc/sysconfig/nfs
RPCNFSDARGS="-V 4.2"

[root@server0 ~]# systemctl restart nfs-server
[root@server0 ~]# systemctl restart nfs-secure-server
[root@server0 ~]# systemctl enable nfs-secure-server


# 客户端

# 下载key文件
[root@desktop0 ~]# wget http://classroom.example.com/pub/keytabs/desktop0.keytab -O /etc/krb5.keytab
[root@desktop0 ~]# mkdir /mnt/nfssecure

[root@desktop0 ~]# vim /etc/fstab
server0:/nfs2   /mnt/nfssecure  nfs     defaults,v4.2,sec=krb5p 0 0

[root@desktop0 ~]# systemctl restart nfs-secure
[root@desktop0 ~]# systemctl enable  nfs-secure

[root@desktop0 ~]# mount -a
[root@desktop0 ~]# df -Th
文件系统                类型      容量  已用  可用 已用% 挂载点
devtmpfs                devtmpfs  1.4G     0  1.4G    0% /dev
tmpfs                   tmpfs     1.4G     0  1.4G    0% /dev/shm
tmpfs                   tmpfs     1.4G  9.4M  1.4G    1% /run
tmpfs                   tmpfs     1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        56G  1.4G   55G    3% /
/dev/sda1               xfs       497M  143M  355M   29% /boot
tmpfs                   tmpfs     283M     0  283M    0% /run/user/0
//server0/smb2          cifs       56G  1.4G   55G    3% /mnt/smb2
server0:/nfs1           nfs4       56G  1.4G   55G    3% /mnt/nfs1
server0:/nfs2           nfs4       56G  1.4G   55G    3% /mnt/nfssecure

```

###  网络链路聚合

```shell




```

###  firewalld 控制port

```shell
配置server0端口转发,从172.25.10.0/24网段访问server0端口6666/tcp时,转发到80/tcp

[root@server0 ~]# firewall-cmd --permanent --add-rich-rule=' rule family="ipv4" source address="172.25.10.0" forward-port port="6666" protocol="tcp" to-port="80"'
success
[root@server0 ~]# firewall-cmd --reload
success
[root@server0 ~]# firewall-cmd --permanent --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: dhcpv6-client nfs samba ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="172.24.3.0/24" service name="ssh" reject
	rule family="ipv4" source address="172.25.10.0" forward-port port="6666" protocol="tcp" to-port="80"


```

###  Iscsi服务器

#####  服务端安装配置

```shell
# 服务端

# 准备共享卷
[root@server0 ~]# ll /dev/sdb2
brw-rw----. 1 root disk 8, 18 9月  27 14:16 /dev/sdb2

#安装软件包
[root@server0 ~]# yum -y install targetcli
[root@server0 ~]# systemctl enable target
Created symlink from /etc/systemd/system/multi-user.target.wants/target.service to /usr/lib/systemd/system/target.service.
[root@server0 ~]# systemctl restart  target

# 配置：
[root@server0 ~]# targetcli
targetcli shell version 2.1.53
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
/> /backstores/block create iscsi_store /dev/sdb2
Created block storage object iscsi_store using /dev/sdb2.


```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20210927142005.png)

```shell
# 创建后端卷  iscsi_store
/> /backstores/block create iscsi_store /dev/sdb2
Created block storage object iscsi_store using /dev/sdb2.
/> ls
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20210927142156.png)

```shell
# 创建iscsi的磁盘名称为 iqn.2017-04.com.tianyun:server0
/> /iscsi create  iqn.2017-04.com.tianyun:server0
Created target iqn.2017-04.com.tianyun:server0.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20210927142450.png)

```shell
# 创建访问控制
# 此服务只能被desktop0.example.com访问
/> iscsi/iqn.2017-04.com.tianyun:server0/tpg1/acls create iqn.2017-04.com.tianyun:desktop0
Created Node ACL for iqn.2017-04.com.tianyun:desktop0
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20210927142821.png)

```
/> iscsi/iqn.2017-04.com.tianyun:server0/tpg1/luns create /backstores/block/iscsi_store 
Created LUN 0.
Created LUN 0->0 mapping in node ACL iqn.2017-04.com.tianyun:desktop0

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20210927143013.png)

```shell
# 端口设置
# 提供服务的端口为3260
/> /iscsi/iqn.2017-04.com.tianyun:server0/tpg1/portals create 10.4.7.44:3260


# 若配置错，需要清零时
/> clearconfig confirm=True

# 保存
/> saveconfig 
Last 10 configs saved in /etc/target/backup/.
Configuration saved to /etc/target/saveconfig.json
/> exit
Global pref auto_save_on_exit=true
Last 10 configs saved in /etc/target/backup/.
Configuration saved to /etc/target/saveconfig.json

# 检测端口
[root@server0 ~]# ss -auntpl | grep 3260
tcp    LISTEN     0      256       *:3260                  *:* 


# 防火墙设置
[root@server0 ~]# firewall-cmd --permanent --add-port=3260/tcp 
success

[root@server0 ~]# firewall-cmd --reload
success

```

##### 客户端配置

```shell
配置desktop0 ISCSI 客户端
自动连接 iqn.2017-04.com.tianyun:server0
创建大小为500M的分区,格式化为 ext4文件系统,自动挂载到/mnt/iscsidisk

# 安装软件包
[root@desktop0 ~]# yum -y install iscsi*

[root@desktop0 ~]# vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2017-04.com.tianyun:desktop0


[root@desktop0 ~]# systemctl restart iscsid
[root@desktop0 ~]# systemctl enable iscsid
Created symlink from /etc/systemd/system/multi-user.target.wants/iscsid.service to /usr/lib/systemd/system/iscsid.service.
[root@desktop0 ~]# 

# 发现
[root@desktop0 ~]# iscsiadm  -m discovery -t st -p server0
10.4.7.44:3260,1 iqn.2017-04.com.tianyun:server0

#连接
[root@desktop0 ~]# systemctl restart iscsi
[root@desktop0 ~]# systemctl enable iscsi
[root@desktop0 ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   60G  0 disk 
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 59.5G  0 part 
  ├─centos-root 253:0    0 55.5G  0 lvm  /
  └─centos-swap 253:1    0    4G  0 lvm  [SWAP]
sdb               8:16   0    2G  0 disk 
sr0              11:0    1  4.4G  0 rom  

# 创建大小为500M的分区,格式化为 ext4文件系统,自动挂载到/mnt/iscsidisk
[root@desktop0 ~]# fdisk /dev/sdb
欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。

Device does not contain a recognized partition table
使用磁盘标识符 0x379a5bb6 创建新的 DOS 磁盘标签。

命令(输入 m 获取帮助)：n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
分区号 (1-4，默认 1)：
起始 扇区 (8192-4194303，默认为 8192)：
将使用默认值 8192
Last 扇区, +扇区 or +size{K,M,G} (8192-4194303，默认为 4194303)：+500M
分区 1 已设置为 Linux 类型，大小设为 500 MiB

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.
正在同步磁盘。

[root@desktop0 ~]# partprobe  /dev/sdb
[root@desktop0 ~]# ll /dev/sdb*
brw-rw----. 1 root disk 8, 16 9月  27 15:28 /dev/sdb
brw-rw----. 1 root disk 8, 17 9月  27 15:28 /dev/sdb1

# 格式化
[root@desktop0 ~]# mkfs.ext4 /dev/sdb1

# 创建挂载点
[root@desktop0 ~]# mkdir /mnt/iscsidisk

# 挂载时必须使用UUID
[root@desktop0 ~]# blkid
/dev/sda1: UUID="4f3ddeb9-d23a-4cc5-a0b8-fe61e93c1c4a" TYPE="xfs" 
/dev/sda2: UUID="YvUdNI-fEv9-qWIH-1rrZ-BGKQ-bhBm-QDEeQ4" TYPE="LVM2_member" 
/dev/sr0: UUID="2019-09-11-18-50-31-00" LABEL="CentOS 7 x86_64" TYPE="iso9660" PTTYPE="dos" 
/dev/mapper/centos-root: UUID="2e2e2e63-0050-4562-a0e1-3f5ac6379be5" TYPE="xfs" 
/dev/mapper/centos-swap: UUID="6e89599a-8cd1-493d-bfb9-6e064d596d38" TYPE="swap" 
/dev/sdb1: UUID="35149673-4fa2-4bfa-a112-c838028649ef" TYPE="ext4" 

# 挂载
[root@desktop0 ~]# vim /etc/fstab
UUID="35149673-4fa2-4bfa-a112-c838028649ef"  /mnt/iscsidisk     ext4    defaults,_netdev 0 0
# _netdev  非本地设备，在网络环境下挂载
 
[root@desktop0 ~]# mount -a
[root@desktop0 ~]# df -Th
文件系统                类型      容量  已用  可用 已用% 挂载点
devtmpfs                devtmpfs  1.4G     0  1.4G    0% /dev
tmpfs                   tmpfs     1.4G     0  1.4G    0% /dev/shm
tmpfs                   tmpfs     1.4G  9.4M  1.4G    1% /run
tmpfs                   tmpfs     1.4G     0  1.4G    0% /sys/fs/cgroup
/dev/mapper/centos-root xfs        56G  1.4G   55G    3% /
/dev/sda1               xfs       497M  143M  355M   29% /boot
tmpfs                   tmpfs     283M     0  283M    0% /run/user/0
//server0/smb2          cifs       56G  1.5G   55G    3% /mnt/smb2
server0:/nfs1           nfs4       56G  1.5G   55G    3% /mnt/nfs1
server0:/nfs2           nfs4       56G  1.5G   55G    3% /mnt/nfssecure
/dev/sdb1               ext4      477M  2.3M  445M    1% /mnt/iscsidisk

```

### Apache服务器

```shell
# 安装必要的软件包
[root@server0 ~]# yum -y install httpd mod_ssl mod_wsgi

# 防火墙设置
[root@server0 ~]# firewall-cmd --permanent --add-service=http
success
[root@server0 ~]# firewall-cmd --permanent --add-service=https
success
[root@server0 ~]# firewall-cmd --reload
success


# 下载网站首页
[root@server0 ~]# wget http://www.baidu.com/index.html -O /var/www/html/index.html

[root@server0 ~]# cd /etc/httpd/conf.d
[root@server0 conf.d]# vim www0.example.com.conf

<VirtualHost *:80>
   ServerName www0.example.com
   DocumentRoot /var/www/html
</VirtualHost>

<Directory "/var/www/html">
  <RequireAll>
        Require all granted
        Require not ip 172.24.3.0/24
  </RequireAll>
</Directory>


[root@server0 conf.d]# systemctl restart httpd
[root@server0 conf.d]# systemctl enable  httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.

# 测试

```

