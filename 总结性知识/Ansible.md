# Ansible

**学习资源：https://galaxy.ansible.com/         通过该网站可以获取全球大佬的roles文档，可以根据自己的所需使用**

## 环境部署

### 1、环境

```shell
环境：
[root@ansible ~]# vim /etc/hosts
10.4.7.44 web01
10.4.7.55 web02
10.4.7.11 ansible

```

### 2、 安装epel源

```shell
[root@ansible ~]# wget https://mirror.tuna.tsinghua.edu.cn/epel/7/x86_64/Packages/e/epel-release-7-14.noarch.rpm

[root@ansible ~]# yum -y localinstall epel-release-7-14.noarch.rpm 
```

### 3、配置ssh公钥

```shell
[root@ansible ~]# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:nVTvH0jXfZNsI6fhIp4DxgjE72Z50C4b7+45FuNjLcs root@ansible
The key's randomart image is:
+---[RSA 2048]----+
|  ..        .    |
|  ..       . .. +|
|   .. .   .  +.O+|
|    .oo. o .o.O +|
|    ..++S.o. +.. |
|     B.=o o .  ..|
|    o B ++      .|
|     ..O...      |
|      *E=        |
+----[SHA256]-----+
[root@ansible ~]# cd ~/.ssh/
[root@ansible .ssh]# ll
总用量 8
-rw-------. 1 root root 1675 11月 15 14:02 id_rsa
-rw-r--r--. 1 root root  394 11月 15 14:02 id_rsa.pub
[root@ansible .ssh]# ssh-copy-id 10.4.7.44
[root@ansible .ssh]# ssh-copy-id 10.4.7.55
```

### 4、安装ansible

```shell
[root@ansible ~]# yum -y install ansible
[root@ansible ~]# ansible --version
ansible 2.9.25
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Aug  7 2019, 00:51:29) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]

```

## ansible基础

### 1、介绍

* ansible是自动化运维工具，基于Python开发，分布式，无序客户端，轻量级，可以实现批量操作，
* ansible其本身不具有批量部署的能力，真正具有批量部署能力的是其模块，ansible只是提供一种框架

### 2、特性

* no agents：不需要在被管控主机上安装任何客户端,更新时,只需在操作机上进行一次更新即可（不用安装客户端。分布式的）
* no server：无服务器端,使用时直接运行命令即可
* modules in any languages：基于模块工作,可使用任意语言开发模块
* yaml,not code：使用yaml语言定制剧本playbook
* ssh by default：基于SSH工作
* strong multi-tier solution：可实现多级指挥

### 3、ansible执行流程

* 读取ansible.cfg文件
* 通过规则过滤inventory中定义的主机列表
* 加载task对应的模块
* 通过ansible core 将模块或命令打包成python脚本文件
* 将临时脚本文件传输到远程服务器
* 对应执行用户家目录的‘ansible/tmp/xx/xxx.py’文件
* 给文件加执行权限
* 执行py文件并返回结果
* 删除临时文件并退出

### 4、配置文件

* 主机清单文件 `/etc/ansible/hosts`
* 主配置文件 `/etc/ansible/ansible.cfg`
* 语法：`ansible  <pattern>  -m <module_name>  -a  <arguments>`
  * pattern--主机清单里定义的主机组名,主机名,IP,别名等,all表示所有的主机,支持通配符,正则
  * :           --多个组,组名之间用冒号隔开
  * *web*    --组名或主机名中含web的
  * webservers[0] - webservers组中的第一台主机
  * 以~开头,匹配正则
  * -m module_name: 模块名称,默认为command
  * -a arguments: 传递给模块的参数

## ansible组件--inventory

* inventory文件通常用于定义要管理主机及其认证信息，例如ssh登录用户名、密码以及key相关信息

```shell
#vim /etc/ansible/hosts
web1                  //单独指定主机，可以使用主机名称或IP地址
[webservers]        //使用[]标签指定主机组
#-----------------------------------------------------------------
为每个主机单独指定变量，这些变量随后可以在 playbooks 中使用：内置变量
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
#------------------------------------------------------------------
为一个组指定变量，组内每个主机都可以使用该变量：
[atlanta:vars]
ansible_ssh_pass='123'
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
#-----------------------------------------------------------------
组可以包含其他组：
[southeast:children]   //southeast包括两个子组
atlanta
raleigh
```

### 1、主机参数

```shell
ansible基于ssh连接inventory中指定的远程主机时，还可以通过参数指定其交互方式:
    ansible_ssh_host # 远程主机
    ansible_ssh_port # 指定远程主机ssh端口
    ansible_ssh_user # ssh连接远程主机的用户,默认root
    ansible_ssh_pass # 连接远程主机使用的密码,在文件中明文,建议使用--ask-pass或者使用SSH keys
    ansible_sudo_pass # sudo密码, 建议使用--ask-sudo-pass
    ansible_connection # 指定连接类型: local, ssh, paramiko
    ansible_ssh_private_key_file # ssh 连接使用的私钥
    ansible_shell_type # 指定连接对端的shell类型, 默认sh,支持csh,fish
    ansible_python_interpreter # 指定对端使用的python编译器的路径
```

###  2、查看组内主机列表

```shell
语法：ansible  组名  --list-hosts
[root@ansible ~]# ansible web --list-hosts
  hosts (2):
    10.4.7.44
    10.4.7.55

[root@ansible ~]# ansible web* -m  ping -o
10.4.7.55 | SUCCESS => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "ping": "pong"}
10.4.7.44 | SUCCESS => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "ping": "pong"}
[root@ansible ~]# ansible web* -m shell -a 'uptime'
10.4.7.55 | CHANGED | rc=0 >>
 10:12:44 up 21:01,  3 users,  load average: 0.01, 0.02, 0.05
10.4.7.44 | CHANGED | rc=0 >>
 10:12:44 up 21:00,  3 users,  load average: 0.00, 0.01, 0.05
[root@ansible ~]# ansible web* -m shell -a 'uptime' -o
10.4.7.55 | CHANGED | rc=0 | (stdout)  10:12:51 up 21:01,  3 users,  load average: 0.01, 0.02, 0.05
10.4.7.44 | CHANGED | rc=0 | (stdout)  10:12:51 up 21:00,  3 users,  load average: 0.00, 0.01, 0.05

```

### 3、注意

* 一个典型的例子就是 shell 和 command 模块. 这两个模块在很多情况下都能完成同样的工作， 以下是两个模块之前的区别：
  * command 模块命令将不会使用 shell 执行. 因此, 像 $HOME 这样的变量是不可用的。还有像 |,& 都将不可用。
  * shell 模块通过shell程序执行， 默认是/bin/sh, <, >, |, ;, & 可用。

​     

## ansible组件--ad-hoc

*  ad-hoc	（点对点）临时的，就是执行一条简单的命令    对于复杂的命令则有playbook
* 帮助文档：
  * -l	获取列表
  * -s  module_name    获取指定模块的使用信息
  * -f   指定要使用的并行进程数，默认为5个，临时指定，如果想要永久的指定，需要修改主配置文件
  * `[root@ansible-server ~]# ansible-doc -l`

### 常用模块

#### 1、远程复制备份模块：copy

**参数**

* src=：指定源文件路径
* dest=：目标地址（拷贝到哪里）
* owner：指定属主
* group：指定属组
* mode：指定权限
* backup：在覆盖之前将源文件备份

```shell
[root@ansible ~]# echo "123456 ----hualaotou " >jiajia.txt
[root@ansible ~]# ansible web01 -m copy -a 'src=/root/jiajia.txt dest=/opt owner=root group=root mode=644' -o
web01 | CHANGED => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": true, "checksum": "1a149e91e621a6cbff9cb9f8f689690e5a48e7f4", "dest": "/opt/jiajia.txt", "gid": 0, "group": "root", "md5sum": "36fc30658d36eb5e733f7acd1cb7de08", "mode": "0644", "owner": "root", "secontext": "system_u:object_r:usr_t:s0", "size": 22, "src": "/root/.ansible/tmp/ansible-tmp-1637032013.36-13799-17830002365049/source", "state": "file", "uid": 0}
[root@web01 opt]# ll
总用量 4
-rw-r--r--. 1 root root 22 11月 16 11:06 jiajia.txt
[root@web01 opt]# cat jiajia.txt 
123456 ----hualaotou 

# 相关选项如下：
    backup：在覆盖之前，将源文件备份，备份文件包含时间信息。有两个选项：yes|no
    content：用于替代“src”，可以直接设定指定文件的值
    dest：必选项。要将源文件复制到的远程主机的绝对路径，如果源文件是一个目录，那么该路径也必须是个目录
    directory_mode：递归设定目录的权限，默认为系统默认权限
    force：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当目标主机的目标位置不存在该文件时，才复制。默认为yes
    others：所有的file模块里的选项都可以在这里使用
    src：被复制到远程主机的本地文件，可以是绝对路径，也可以是相对路径。如果路径是一个目录，它将递归复制。在这种情况下，如果路径使用“/”来结尾，则只复制目录里的内容，如果没有使用“/”来结尾，则包含目录在内的整个内容全部复制，类似于rsync
```

#### 2、用户管理user模块

* 添加用户并设置密码，使用命令需要双引号

```shell
# 添加用户
[root@ansible ~]# ansible web01 -m user -a "name=jiajia password=123456" -o
# 删除用户
[root@ansible ~]# ansible web01 -m user -a "name=jiajia state=absent" -o
# state=absent  删除用户，但是不会删除家目录

user模块
home：指定用户的家目录，需要与createhome配合使用
groups：指定用户的属组
uid：指定用的uid
password：指定用户的密码
name：指定用户名
createhome：是否创建家目录 yes|no
system：是否为系统用户
remove：当state=absent时，remove=yes则表示连同家目录一起删除，等价于userdel -r
state：是创建还是删除
shell：指定用户的shell环境
```

#### 3、软件包管理  yum模块

```shell
# 安装Apache
[root@ansible ~]# ansible web02 -m yum -a "name=httpd state=latest" -o
state=     #状态是什么，干什么
state=absent        用于remove安装包
state=latest       表示最新的
state=removed      表示卸载
# 卸载Apache
[root@ansible ~]# ansible web02 -m yum -a "name=httpd state=removed" -o

# 其选项有： 
config_file：yum的配置文件 
disable_gpg_check：关闭gpg_check 
disablerepo：不启用某个源 
enablerepo：启用某个源
name：要进行操作的软件包的名字，也可以传递一个url或者一个本地的rpm包的路径 
state：状态（present，absent，latest）
```

#### 4、服务管理service模块

```shell
[root@ansible ~]# ansible web02 -m yum -a "name=httpd state=started enabled=yes" -o
# state=started   启动
# state=stopped	停止
# state=restarted		重启
# enabled=yes		开机自启
# enabled=no		开机关闭

该模块包含如下选项： 
arguments：给命令行提供一些选项 
enabled：是否开机启动 yes|no
name：必选项，服务名称 
pattern：定义一个模式，如果通过status指令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务依然在运行
runlevel：运行级别
sleep：如果执行了restarted，在则stop和start之间沉睡几秒钟
state：对当前服务执行启动，停止、重启、重新加载等操作（started,stopped,restarted,reloaded）
```

#### 5、文件模块file

```shell
# 创建文件
[root@ansible ~]# ansible web01 -m file -a 'path=/tmp/99.txt mode=777 state=touch' -o
owner:修改属主
group:修改属组
mode:修改权限
path=:要修改文件的路径
recurse：递归的设置文件的属性，只对目录有效
        yes:表示使用递归设置
state:
touch:创建一个新的空文件
directory:创建一个新的目录，当目录存在时不会进行修改

# 相关选项如下：
    force：需要在两种情况下强制创建软链接，一种是源文件不存在，但之后会建立的情况下；另一种是目标软链接已存在，需要先取消之前的软链，然后创建新的软链，有两个选项：yes|no
    group：定义文件/目录的属组
    mode：定义文件/目录的权限
    owner：定义文件/目录的属主
    path：必选项，定义文件/目录的路径
    recurse：递归设置文件的属性，只对目录有效，有两个选项：yes|no
    src：被链接的源文件路径，只应用于state=link的情况
    dest：被链接到的路径，只应用于state=link的情况
    state：
           directory：如果目录不存在，就创建目录
           file：即使文件不存在，也不会被创建
           link：创建软链接
           hard：创建硬链接
           touch：如果文件不存在，则会创建一个新的文件，如果文件或目录已存在，则更新其最后修改时间
           absent：删除目录、文件或者取消链接文件
```

#### 6、收集信息模块setup

```shell
[root@ansible ~]# ansible web01 -m setup -a 'filter=ansible_all_ipv4_addresses'
web01 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.4.7.44"
        ], 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}

filter:过滤
```

#### 7、任务管理 cron模块

```shell
用于管理计划任务
包含如下选项： 
backup：对远程主机上的原任务计划内容修改之前做备份 
cron_file：如果指定该选项，则用该文件替换远程主机上的cron.d目录下的用户的任务计划 
day：日（1-31，*，*/2,……） 
hour：小时（0-23，*，*/2，……）  
minute：分钟（0-59，*，*/2，……） 
month：月（1-12，*，*/2，……） 
weekday：周（0-7，*，……）
job：要执行的任务，依赖于state=present 
name：该任务的描述 
special_time：指定什么时候执行，参数：reboot,yearly,annually,monthly,weekly,daily,hourly 
state：确认该任务计划是创建还是删除 
user：以哪个用户的身份执行

ansible test -m cron -a 'name="a job for reboot" special_time=reboot job="/some/job.sh"'
```

#### 8、shell命令

```shell
ansible默认使用的模块是command,支持多数shell命令,但不支持shell变量及管道,如果要使用,用shell模块
```

#### 9、ping测试

```shell
ping: 测试远程主机的运行状态
```

## playbook

* playbook是ansible用于配置，部署，和管理被控制节点的剧本
* playbook是有一个或多个play组成的列表
* playbook由YMAL语言编写，YMAL格式类似于JSON文件格式，便于理解和书写

### 1、核心元素

* Variables     #变量元素,可传递给Tasks/Templates使用; 
* Tasks          #任务元素,由模块定义的操作的列表，即调用模块完成任务; 
* Templates   #模板元素,使用了模板语法的文本文件，可根据变量动态生成配置文件;
* Handlers     #处理器元素,通常指在某事件满足时触发的操作;
* Roles          #角色元素
* **注意       一个剧本里面可以有多个play，每个play只能有一个tasks，每个tasks可以有多个name**

### 2、playbook的基础组件

* name:
  * 定义playbook或者task的名称（描述信息），每一个play都可以完成一个任务
* hosts:
  * hosts用于指定要执行指定任务的主机，可以是一个或多个由冒号分割的主机组
* user:
  * 指定远程主机上的执行任务的用户，也可以使用remote_user（基本上是root）
* tasks:
  * 任务列表play的主体部分是task list，task list中的各任务按次序逐个在hosts中指定的所有的主机上执行，即所有主机上完成第一个任务后在开始第二个
* vars:
  * 定义变量
* vars_files:
  * 调用定义变量文件
* notify:
  * 用于当前股关注的资源发生变化时采取一定指定的操作
* handlers:
  * 能包含的包括task，handler和playbook
  * 可以再include的时候传递变量

### 3、案例

**案例一**

```shell
[root@ansible ~]# cd /etc/ansible/
[root@ansible ansible]# vim test.yml
 - hosts: web01
   user: root
   tasks:
   - name: playbook_test
     file: state=touch path=/tmp/playbook.txt
     
# 检测语法
[root@ansible ansible]# ansible-playbook --syntax-check test.yml

playbook: test.yml
[root@ansible ansible]# 

#运行playbook
[root@ansible ansible]# ansible-playbook test.yml
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211117170811.png)

**案例二**

```shell
handlers：由特定条件触发的Tasks
语法：
tasks：
- name: TASK_NAME
  module: arguments              #1.上面任务执行成功，然后
  notify: HANDLER_NAME        #2.通知他
handlers:
- name: HANDLER_NAME        #3.一一对应，这里的描述与notify定义的必须一样
  module: arguments              #4.执行这个命令
  
[root@ansible ansible]# vim handlers.yml
- hosts: web01
  user: root
  tasks:
  - name: test copy
    copy: src=/root/jiajia/hualaotou dest=/jiajia
    notify: test handlers
  handlers:
  - name: test handlers
    shell: echo "曹操" >> /jiajia/hualaotou
    
[root@ansible ansible]# ansible-playbook --syntax-check handlers.yml 

playbook: handlers.yml
[root@ansible ansible]# ansible-playbook handlers.yml

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211118094808.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211118094856.png)

**注意：          只有 copy 模块真正执行后，才会去调用下面的 handlers 相关的操作，追加内容。所以这种比较适合配置文件发生更改后，需要重启服务的操作。**

**案例三**

```shell
循环：迭代，需要重复执行的任务；
对迭代项的引用，固定变量名为”item”，使用with_item属性给定要迭代的元素； 
元素：1.列表 2.字符串 3.字典
[root@ansible ansible]# vim list.yml
- hosts: web01
  remote_user: root
  tasks:
  - name: install packages
    yum: name={{ item }} state=latest         #相当于for循环里面的i 
    with_items:                                                #取值 。但是不支持通配符
     - httpd
     - php
     - php-mysql
     - php-mbstring
     - php-gd 
     
基于字典列表给元素示例：     
[root@ansible ansible]# vim list1.yml
- hosts: all
  user: root
  tasks:
  - name: create groups
    group: name={{ item }} state=present
    with_items:
    - groupx1
    - groupx2
    - groupx3
   - name: create users
     user: name={{ item.name }} group={{ item.group }} state=present
     with_items:           #这里使用的是字典
     - {name: 'userx1', group: 'groupx1'}
     - {name: 'userx2', group: 'groupx2'}
     - {name: 'userx3', group: 'groupx3'}

```

**案例四**

```shell
tags使用：
        给指定的任务定义一个调用标识，形式如下
        只运行指定标记的任务：-t tags
        [root@ansible ansible]# ansible-playbook  -t 标记名称 test.yml 

        跳过某一个被标记的任务：--skip-tags=SKIP_TAGS 
        [root@ansible ansible]# ansible-playbook  --skip-tags=标记名称 test.yml 

        从某一个任务开始往下运行：--start-at-task 任务名称
        [root@ansible ansible]# ansible-playbook  --start-at-task "start httpd service" test.yml

[root@ansible ansible]# vim tag.yml
---
 - hosts: web
   user: root
   tasks:
   - name: touch file
     file: path=/root/nihao.txt state=touch
     tags: nihao
   - name: mkdir file
     file: path=/root/xingdian state=directory
     tags: xingdian
   - name: touch file1
     file: path=/root/file1.txt state=touch
     tags: file1
```

### 4、变量

#### 1、变量调用

```shell
{{ var_name }}

# 示例：
    yum: name={{ item }} state=latest         #相当于for循环里面的i 
    with_items:                                                #取值 。但是不支持通配符
     - httpd
     - php
     - php-mysql
```

#### 2、内建变量

**由facts组件提供，可以使用setup模块查询**

```shell
[root@ansible ansible]# ansible web01 -m setup | grep "mem"
        "ansible_memfree_mb": 2156, 
        "ansible_memory_mb": {
        "ansible_memtotal_mb": 2827,
```

#### 3、自定义变量

```shell
1.用命令行传递参数
为了使Playbook更灵活、通用性更强，允许用户在执行的时候传入变量的值，这个时候就需要用到“额外变量”。
定义命令行变量
在release.yml文件里，hosts和user都定义为变量，需要从命令行传递变量值。
案例：
[root@ansible ansible]# vim e33_var_in_command.yml
---
 - hosts: '{{ hosts }}'
   user: '{{ user }}'
   tasks:
   - name: new file
     file: path=/root/var1.txt state=touch
# 使用命令行变量
# 在命令行里面传值的方法：  
ansible-playbook e33_var_in_command.yml --extra-vars "hosts=web user=root"
# 还可以用json格式传递参数：  
ansible-playbook e33_var_in_command.yml --extra-vars "{'hosts':'vm-rhel7-1', 'user':'root'}"
2、set_fact自定义facts变量：
  set_fact模块可以自定义facts，这些自定义的facts可以通过template或者变量的方式在playbook中使用。如果你想要获取一 个进程使用的内存的百分比，则必须通过set_fact来进行计算之后得出其值，并将其值在playbook中引用。
```

## role

**role是在ansible中，playbook的目录组织结构**

每一个角色是有名字的，他是一个目录，可以包含子目录

以特定的层级目录结构进行组织的tasks、variables、handlers、templates、files等：

* **role_name**：这个是角色的名称
* **file**：存储有copy或script等模块调用的文件
* **tasks**：专门存储任务的目录，一个角色可以定义多个任务
  * 此目录中至少应该有一个名为main.yml的文件，用于定义个task；其他的文件需要由main.yml进行包含调用
* **handlers**：条件     前一个任务执行成功去执行下面的，处理特定事物的文件
  * 此目录中至少应该有一个名为main.yml的文件，用于定义各handler；其他的文件需要由main.yml进行包含调用
* **vars**：  变量     定义变量的文件
  * 此目录中至少应该有一个名为main.yml的文件，用于定义各variable；其他的文件需要由main.yml进行包含调用
* **templates**：  模板   使用变量的文件存储由template模块调用的模板文本
* **meta**：  此目录中至少应该有一个名为main.yml的文件，定义当前角色的特殊设定及其依赖关系，其他的文件需要由main.yml进行包含调用
* **default**：此目录中至少应该有一个名为main.yml 的文件，用于设定默认变量

### nginx案例

**1、准备目录结构**

```shell
[root@ansible ~]# cd /etc/ansible/roles/ 
[root@ansible roles]# mkdir nginx/{files,handlers,tasks,templates,vars} -p
```

**2、创建文件**

```shell
[root@ansible roles]# touch site.yml nginx/{handlers,tasks,vars}/main.yml
[root@ansible roles]# yum install -y tree
[root@ansible roles]#  tree nginx/
nginx/
├── files
├── handlers
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
└── vars
    └── main.yml

5 directories, 3 files

```

**3、创建nginx的测试文件**

```shell
[root@ansible roles]# echo "12345678910" > nginx/files/index.html
```

**4、安装nginx并配置模板**

```shell
[root@ansible roles]# yum install -y nginx && cp /etc/nginx/nginx.conf nginx/templates/nginx.conf.j2

# 编写任务
[root@ansible roles]# vim nginx/tasks/main.yml
---
- name: install epel
  yum: name=epel-release state=latest
- name: install nginx
  yum: name=nginx state=latest
- name: copy nginx.conf templte
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
- name: copy index.html
  copy: src=index.html dest=/usr/share/nginx/html/index.html
  notify: start nginx    

```

**5、templates模板**

```shell
[root@ansible roles]# vim nginx/templates/nginx.conf.j2
......
user nginx;
# worker_processes auto;
worker_processes {{ worker_connections }};
......
```

**6、编写变量**

```shell
[root@ansible roles]# vim nginx/vars/main.yml
worker_connections: 2
```

**7、编写handlers**

```shell
[root@ansible roles]# vim nginx/handlers/main.yml
---
- name: start nginx
  service: name=nginx state=started

```

**8、编写剧本**

```shell
[root@ansible roles]# vim site.yml
---
- hosts: web02
  user: root
  roles:
   - nginx

[root@ansible ansible]# tree roles/
roles/
├── nginx
│   ├── files
│   │   └── index.html
│   ├── handlers
│   │   └── main.yml
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   │   └── nginx.conf.j2
│   └── vars
│       └── main.yml
└── site.yml

6 directories, 6 files

```

**9、检测语法**

```shell
[root@ansible roles]# ansible-playbook site.yml --syntax-check

playbook: site.yml
```

**10、执行剧本**

```shell
[root@ansible roles]# ansible-playbook site.yml
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211118143041.png)

**11、查看**

```shell
[root@web02 ~]# ss -auntpl | grep nginx

[root@web02 ~]# cat /etc/nginx/nginx.conf | grep pro
# worker_processes auto;
worker_processes 2;
```

**12、访问**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211118143438.png)

#  视频学习

## 大纲

* 什么是ansible
* ansible的特性\优点
* ansible的基础架构（控制端\被控端\invventory\ad-hoc\playbook\连接协议。。。。）
* ansible安装
* ansible配置
* ansible   inventory
* ansible  ad-hoc   shell命令
* ansible  playbook   shell脚本    yaml
* 变量    variables
  * 变量优先级
* 判断语句
* 循环语句
* 异常处理
* tag标签
* handlers  触发器
* include 包含
* ansible  jinja模板
  * keepalived
  * nginx _proxy
* ansible  role角色
  * 编配工具-----清晰目录规划----- 严格按照目录规划来
* ansible   galaxy
* ansible   tower
* ansible部署集群架构

**ansible是一个IT自动化配置管理工具**

### ansible可以完成哪些功能？

* 批量执行远程命令
* 批量配置软件服务
* 实现软件开发功能
* 编排高级的IT任务

###  ansible的特点

* 容易学习、无代理模式
* 操作灵活
* 简单易用
* 安全可靠
* 移植性高

### ansible配置文件存在优先级问题

* ansible.cfg        	项目目录
* .ansible.cfg              当前用户的家目录
* /etc/ansible/ansible.cfg      

###  ansible配置文件相关信息

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122094403.png)

###  ansible 管理机下发公钥

`sshpass      -p   password     ssh-copy-id   -i   ~/.ssh/id_rsa.pub    root@ip`

####  若想返回值是别名

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122095440.png)

**生产案例：**

如果控制端和被控制端第一次通讯，需要确认指纹信息，如果机器特别多的情况下怎么办？

​	将ansible 配置文件中的  host_key_checking = False  参数注释打开即可

​	但是要注意ansible.cfg的读取顺序

###  使用ansible执行一次远程命令，注意观察返回结果的颜色

* 绿色：代表被管理端主机没有被修改
* 黄色：代表被管理端主机发生变更
* 红色：代表出现了故障，注意查看提示

## ad-hoc

### ad-hoc模式的命令使用

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122101301.png)

###  ad-hoc模式常用的模块

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122102352.png)

**command  shell 本质上执行都是基础命令   （command不支持管道技术）**

###  查看模块的参数

```shell
[root@ansible ~]# ansible-doc yum
[root@ansible ~]# ansible-doc   -s   yum
```

### yum模块示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122105126.png)

### copy模块示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122110820.png)

**copy和template的区别：**

**copy是将文件原样拷贝**

**template会将拷贝的文件进行变量的解析，然后在分发**

### get_url模块示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122111413.png)

### file模块示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122112130.png)

###  service模块示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122112545.png)

### group模块示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122113926.png)

### user模块示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122134643.png)

### cron模板示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122135928.png)

### mount 模板示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122144244.png)

### selinux模板示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122145311.png)

### firewalld模板示例

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122150255.png)

## playbook

###  案例

#### 案例一

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122162515.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122162756.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122162729.png)

#### 案例二

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122162225.png)

#### 案例三

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122163104.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122164314.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122164413.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122164440.png)

### 变量

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211122175246.png)

####  1、在playbook中定义

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123093412.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123094453.png)

#### 2、在inventory主机清单中定义

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123095505.png)

**注意：**

​		默认情况下，group_vars目录中的文件名与hosts清单中的组名保持一致

​		比如在group_vars目录中创建了oldboy组的变量，其他组是无法使用oldboy组的变量

​		系统提供了一个特殊组，all，只需要在group_vars目录下建立一个all文件，编写好变量，所有组都可以使用

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123100420.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123101649.png)

**注意：**

* host_vars      特殊的变量目录，针对单个主机进行变量
* group_vars       特殊的变量目录，针对inventory主机清单中的组进行变量定义，对A组定义的变量   B组无法调用
* group_vars/all        特殊的变量文件，可以针对所有的主机组定义变量

#### 3、通过执行playbook是使用-e参数指定变量

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123103709.png)

####  4、ansible变量优先级

* 1、外置传参
* 2、playbook（vars_files-------> vars）
* 3、inventory（host_vars ----->group_vars/group_name------->group_vars-all）

#### 5、ansible的变量注册

**register      debug**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123110502.png)

####  6、ansible的facts变量

**用来采集被控端的状态指标：比如：ip地址   主机名称    CPU信息   内存等**

**默认情况的facts变量名都已经预先定义好了，只需要采集被控端的信息，然后传递至facts变量即可**



**在写任何新的服务之前，请先手动测试一遍，提取安装的命令、配置文件路径、启动命令等。。。。**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123132439.png)



#### 7、作业

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123132757.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123153804.png)

 ![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123153313.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123153405.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123153654.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123153625.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123153549.png)

### 任务控制

#### 1、条件判断  when

**案例一**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123163044.png)

 **案例二**![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123164432.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123164549.png)

**or:  ** 或

**and:   ** 多个条件同时满足

**案例三**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211123165406.png)

#### 2、循环语句 with_items

**案例一**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124094806.png)

**案例二**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124095206.png)

**案例三**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124095327.png)

**案例四**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124101329.png)

#### 3、触发器  handlers

**notify监控-------> 通知  --------> handlers触发**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124103522.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124103557.png)

**handlers注意事项：**

* 无论多少的task通知了相同的handlers，handlers仅会在所有tasks结束后运行一次
* 只有task发生改变了才会通知handlers，没有改变则不会触发handlers
* 不能使用handlers替代tasks、因为handlers是一个特殊的tasks

#### 4、标签  tags   

**根据指定的标签执行takes ，调试时使用（执行使用-t参数，--skip-tags  忽略该标签）**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124111029.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124111113.png)

#### 5、包含  include

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124113327.png)

**将多个yml文件合成一个文件**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124113652.png)

#### 6、错误忽视  ignore_errors

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124132351.png)

#### 7、错误处理  changed_when

**案例一：强制调用handlers**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124132902.png)

**案例二：关闭changed的状态（确定该tasks不会对被控制端做任何的修改和变更）**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124133359.png)

**案例三：使用changed_when检查tasks任务返回的结果**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124134737.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124134703.png)

## jinja2  了解项

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124141413.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124143009.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124142929.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124151142.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124151219.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124151241.png)

**jinja实现**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124152026.png) 

## roles

#### 1、roles小技巧

* 创建roles目录结构，手动或使用ansible-galaxy  init test roles
* 编写roles的功能，也就是tasks
* 最后playbook引用roles编写好的tasks

```shell
[root@ansible ~]# mkdir project2
[root@ansible ~]# cd project2
[root@ansible project2]# ansible-galaxy init test
- Role test was created successfully
[root@ansible project2]# tree test    # 默认初始的roles目录太过繁杂，一般都是自己创建
test/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml

```

####  案例一：

```shell
[root@ansible project2]# mkdir memcached/{tasks,handlers,templates,vars,files} -pv
mkdir: 已创建目录 "memcached"
mkdir: 已创建目录 "memcached/tasks"
mkdir: 已创建目录 "memcached/handlers"
mkdir: 已创建目录 "memcached/templates"
mkdir: 已创建目录 "memcached/vars"
mkdir: 已创建目录 "memcached/files"

# 先编辑tasks任务
[root@ansible project2]# vim memcached/tasks/main.yml
- name: Installed Memcached Server
  yum: name=memcache state=present

- name: Configure Memcached Server
  template: src=  dest=

- name: Started Memcached Server
  service: name=memcache state=started enabled=yes
  
  


```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211124165416.png)

##  企业实例

一般的编写步骤

* 基础环境       内核升级、仓库安装、基础软件安装
* 基础服务       nginx     PHP    mysql    redis
* 业务引入     
  * WordPress（nginx   PHP   mysql   redis）
  * nginx_proxy（nginx模块） ----->   proxy.conf

### 案例一

#### 1、基础环境

* 关闭防火墙   firewalld   selinux
* 添加base  epel仓库
* 特定主机需要添加特定的仓库源（nginx、mysql、PHP、zabbix、elk.........）
* 安装基础软件包    rsync   nfs-utils    net-tools   lrzsz    unzip   vim   tree......
* 创建统一用户www   uid为666   gid为666
* 内核升级    内核参数调整    文件描述符调整

```shell
# 创建roles目录
[root@ansible ~]# mkdir /opt/roles
[root@ansible ~]# cd /opt/roles
# 先写hosts
[root@ansible roles]# vim hosts
[lbserver]
10.4.7.44
10.4.7.55

[webserver]
10.4.7.21
10.4.7.22
10.4.7.23
10.4.7.24

[nfsserver]
10.4.7.12

[dbserver]
10.4.7.25
10.4.7.26
10.4.7.27

# 分发公钥
[root@ansible roles]# ssh-copy-id -i ~/.ssh/id_rsa.pub  root@10.4.7.12
。。。。
# 测试是否能够连接
[root@ansible roles]# ansible all  -i hosts  -m ping -o

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211125134007.png)

```shell
# 创建项目的ansible.cfg文件
[root@ansible roles]# cp /etc/ansible/ansible.cfg ./
[root@ansible roles]# ansible --version
ansible 2.9.25
  config file = /opt/roles/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Aug  7 2019, 00:51:29) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]


# 创建变量目录
[root@ansible roles]# mkdir host_vars
[root@ansible roles]# mkdir group_vars
[root@ansible roles]# ll
总用量 24
-rw-r--r--. 1 root root 19985 11月 25 13:42 ansible.cfg
drwxr-xr-x. 2 root root     6 11月 25 13:43 group_vars
-rw-r--r--. 1 root root   149 11月 25 13:32 hosts
drwxr-xr-x. 2 root root     6 11月 25 13:43 host_vars


```

#### 2、基础环境yml

```shell
[root@ansible roles]# mkdir base/{tasks,handlers,templates,vars,files} -p
# 编辑任务
[root@ansible roles]# vim base/tasks/main.yml
- name: Disables Firewalld Server
  service: name=firewalld  state=stopped enabled=no

- name: Disabled Selinux Server
  selinux: state=disabled

- name: Create Web {{ web_user }} {{ web_user_id }}Group
  group: name={{ web_user }} gid={{ web_user_id }}

- name: Create Web {{ web_user }} {{ web_user_id }}User
  user: name={{ web_user }} uid={{ web_user_id }} group={{ web_user }}

- name: Add Base Yum Repository
  yum_repository:
    name: base
    description: Base Aliyun Repository
    baseurl: http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
    gpgcheck: yes
    gpgkey: http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

- name: Add Epel Yum Repository
  yum_repository:
    name: epel
    description: Epel Aliyun Repository
    baseurl: http://mirrors.aliyun.com/epel/7/$basearch
    gpgcheck: no

- name: Add Nginx Aliyun Repository
  yum_repository:
    name: nginx
    description: Nginx Repository
    baseurl: http://nginx.org/packages/centos/7/$basearch/
    gpgcheck: no
  when: ( ansible_hostname is match('web*')) or
        ( ansible_hostname is match('lb*'))

- name: Add PHP Yum Repository
  yum_repository:
    name: php71w
    description: php Repository
    baseurl: http://us-east.repo.webtatic.com/yum/el7/x86_64/
    gpgcheck: no
  when: ( ansible_hostname is match('web*'))

- name: Installed Package All
  yum: name={{ packages }} state=present
  vars:
    packages:
      - nfs-utils
      - net-tools
      - wget
      - tree
      - lrzsz
      - vim
      - unzip
      - httpd-tools
      - iftop
      - iotop
      - glances

# /etc/security/limit.conf
- name: Change Limit 
  pam_limits:
    domain: '*'
    limit_type: "{{ item.limit_type }}"
    limit_item: "{{ item.limit_item }}"
    value: "{{ item.value }}"
  with_items:
    - { limit_type: 'soft', limit_item: 'nofile', value: '100000'}
    - { limit_type: 'hard', limit_item: 'nofile', value: '100000'}



# 编辑变量
[root@ansible roles]# vim group_vars/all
web_user: www
web_user_id: 666


# 编辑play
[root@ansible roles]# vim site.yml

- hosts: all
  roles:
    - role: base

# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211125161756.png)

#### 3、服务环境

##### 1、nginx服务

```shell
[root@ansible roles]# mkdir nginx/{tasks,handlers,templates} -p
# 编辑任务
[root@ansible roles]# vim nginx/tasks/main.yml
- name: Installed Nginx Server
  yum: name=nginx  state=present

- name: Configuer Nginx Server
  template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
  notify: Restart Nginx Server

- name: Start Nginx Server
  service: name=nginx  state=started

# 编辑触发器
[root@ansible roles]# vim nginx/handlers/main.yml
- name: Restart Nginx Server
  service: name=nginx  state=restarted


# 编辑模板
[root@ansible roles]# cp /etc/nginx/nginx.conf  nginx/templates/nginx.conf.j2
user  {{ web_user }};
worker_processes  {{ ansible_processor_cores }};
error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  {{ ansible_processor_cores * 2048 }};
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;
    include /etc/nginx/conf.d/*.conf;
}


# 添加至play
[root@ansible roles]# vim site.yml
- hosts: all
  roles:
    - role: base

- hosts: webserver
  roles:
    - role: nginx
      tags: nginx
      
# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml  -t nginx    

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211125165608.png)

##### 2、nfs   服务

```shell
[root@ansible roles]# mkdir nfs/{tasks,templates,handlers} -p
# 编辑tasks任务
[root@ansible roles]# vim nfs/tasks/main.yml
- name:  Install NFS Server
  yum: name=nfs-utils  state=present

- name: Configuer NFS Server
  template: src=exports.j2 dest=/etc/exports
  notify: Restart NFS Server
  
- name: Create NFS Server Share Directory
  file: path={{ nfs_dir }} state=directory owner={{ web_user }} group={{ web_user }}


- name: Started NFS Server
  service: name=nfs  state=started  enabled=yes
  

# 触发器
[root@ansible roles]# vim nfs/handlers/main.yml
- name: Restart NFS Server
  service: name=nfs  state=restarted
  
# 准备模板文件
[root@ansible roles]# cp /etc/exports nfs/templates/exports.j2
{{ nfs_dir }} {{ nfs_share_ip }}(rw,sync,all_squash,anonuid={{ web_user }},anongid={{ web_user_id }})

# 添加变量
[root@ansible roles]# vim group_vars/all
# nfs
nfs_dir: /data
nfs_share_ip: 10.4.7.0/8

# 添加play
[root@ansible roles]# vim site.yml
- hosts: all
  roles:
    - role: base

- hosts: webserver
  roles:
    - role: nginx
      tags: nginx

- hosts: nfsserver
  roles:
    - role: nfs
      tags: nfs

# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml  -t nfs

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126110028.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126110255.png)

##### 3、redis 服务

```shell
[root@ansible roles]# mkdir redis/{tasks,handlers,templates} -p
# 编辑tasks任务
[root@ansible roles]# vim redis/tasks/mail.yml
- name: Install Redis Server
  yum: name=redis state=present

- name: Configuer Redis Server
  template: src=redis.conf.j2 dest=/etc/redis.conf
  notify: Restarted Redis Server


- name: Started Redis Server
  service: name=redis  state=started enabled=yes

# 触发器
[root@ansible roles]# vim redis/handlers/main.yml
- name: Restarted Redis Server
  service: name=redis state=restarted
  
# 模板文件
[root@ansible roles]# cp /etc/redis.conf redis/templates/redis.conf.j2
。。。。。。
bind 127.0.0.1 {{ ansible_default_ipv4.address }}   # 可以使用setup查找变量
。。。。。。

# 添加play
[root@ansible roles]# vim site.yml
- hosts: all
  roles:
    - role: base

- hosts: webserver
  roles:
    - role: nginx
      tags: nginx

- hosts: nfsserver
  roles:
    - role: nfs
      tags: nfs

- hosts: dbserver
  roles:
    - role: redis
      tags: redis

# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml -t redis

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126114127.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126114102.png)

##### 4、mysql 服务

```shell
[root@ansible roles]# mkdir mariadb/{tasks,handlers,templates} -p
# 编辑tasks任务
- name: Install Mariadb Server
  yum: name={{ packages }}  state=present
  vars:
    packages:
      - mariadb
      - mariadb-server
      - MySQL-python

- name: Configuer  Mariadb Server
  template: src=my.cnf.j2  dest=/etc/my.cnf backup=yes
#  notify: Restart Mariadb Server

- name: Started Mariadb Server
  service: name=mariadb  state=started  enabled=yes 

# mariadb创建数据库
- name: Create Application Databases
  mysql_db: name={{ item }} state=present
  with_items:
    - wordpress
    - zh
    - phpmyadmin
    - zabbix

# mariadb 创建用户并授权
- name: Create Web Remote Application DB User
  mysql_user:
    name: "{{ web_db_user }}"
    password: "{{ web_db_pass }}"
    priv: '*.*:ALL'
    host: '%'
    state: present

  
# 触发器
[root@ansible roles]# vim mariadb/handlers/main.yml
- name: Restart Mariadb Server
  service: name=mariadb state=restarted

# 模板文件
[root@ansible roles]# cp /etc/my.cnf  mariadb/templates/my.cnf.j2

# 添加play
- hosts: dbserver
  roles:
    - role: redis
    - role: mariadb
      tags: db
# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml -t db

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126144806.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126152130.png)

##### 5、keepalived 服务

```shell
[root@ansible roles]# mkdir keepalived/{tasks,handlers,templates} -p
[root@ansible roles]# vim keepalived/tasks/main.yml
- name: Install keepalived Server
  yum: name=keepalived state=present

- name: Configure Keepalived Server
  template: src=keepalived.conf.j2 dest=/etc/keepalived/keepalived.conf
  notify: Restart Keepalived Server

- name: Start Keepalived Server
  service: name=keepalived state=started enabled=yes

# 触发器
[root@ansible roles]# vim keepalived/handlers/main.yml
- name: Restart Keepalived Server
  service: name=keepalived state=restarted

# 准备模板
[root@ansible roles]# cp /etc/keepalived/keepalived.conf keepalived/templates/keepalived.conf.j2
! Configuration File for keepalived

global_defs {
   router_id {{ ansible_fqdn }}
}

vrrp_instance VI_1 {
{% if ansible_fqdn == "lbserver_01" %}
    state MASTER
    priority 150
{% elif ansible_fqdn != "lbserver_01" %}
    state BACKUP
    priority 100 
{% endif %}
    interface ens32
    virtual_router_id 51
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.4.7.9
    }

# 添加play
[root@ansible roles]# vim site.yml 
- hosts: lbserver
  roles:
    - role: nginx
    - role: keepalived
  tags: lb


# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml -t lb

```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126163338.png)

##### 6、PHP-fpm 服务

```shell
[root@ansible roles]# mkdir php-fpm/{tasks,handlers,templates} -p
[root@ansible roles]# vim php-fpm/tasks/main.yml
- name: Install PHP-FPM Server
  yum: name={{ packages }} state=present
  vars:
    packages:
      - php71w
      - php71w-cli
      - php71w-common
      - php71w-devel
      - php71w-embedded
      - php71w-gd
      - php71w-mcrypt
      - php71w-mbstring
      - php71w-pdo
      - php71w-xml
      - php71w-fpm
      - php71w-mysqlnd
      - php71w-opcache
      - php71w-pecl-memcached
      - php71w-pecl-redis
      - php71w-pecl-mongodb

- name: configure PHP-FPM Server
  template: src=www.conf.j2  dest=/etc/php-fpm.d/www.conf
  notify: Restart PHP-FPM Server

- name: configure PHP-INI Server
  template: src=php.ini.j2  dest=/etc/php-ini
  notify: Restart PHP-FPM Server

- name: Start PHP-FPM Server
  service: name=php-fpm state=started enabled=yes

# 触发器
[root@ansible roles]# vim php-fpm/handlers/main.yml
- name: Restart PHP-FPM Server
  service: name=php-fpm state=restarted

# 配置模板
[root@ansible roles]# vim php-fpm/templates/www.conf.j2
[www]
user = {{ web_user }}
group = {{ web_user }}
listen = 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
slowlog = /var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache


[root@ansible roles]# vim php-fpm/templates/php.ini.j2
session.save_handler = redis
session.save_path = "tcp://{{ redis_server_ip }}:{{ 
}}"


# 添加变量
[root@ansible roles]# vim group_vars/all
# Redis
redis_server_ip: 10.4.7.26
redis_server_port: 6379


# 添加play
[root@ansible roles]# vim site.yml 
- hosts: webserver
  roles:
    - role: nginx
    - role: php-fpm
  tags: web

# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml -t web


```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126173358.png)

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211126173547.png)

#### 4、业务引入

##### WordPress引入

```shell
[root@ansible roles]# mkdir wordpress/{tasks,handlers,templates,meta，files} -p
# 添加依赖
[root@ansible roles]# vim wordpress/meta/main.yml
dependencies:
- role: nginx
- role: php-fpm

[root@ansible roles]# vim wordpress/tasks/main.yml
- name: Wordpress Virt Server
  template: src=blog.com.j2   dest=/etc/nginx/conf.d/blog.com.conf
  notify: Restart Nginx server

- name: Create Wordpress Data Directory
  file: path={{ wordpress_root_path }} state=directory owner={{ web_user }} group={{ web_user }}

- name: Unzip Wordpress Code
  unarchive: src=wordpress-5.8.2-zh_CN.zip dest={{ wordpress_root_path }} owner={{ web_user }} group={{ web_user }}



# 准备配置文件
[root@ansible roles]# vim wordpress/templates/blog.com.conf.j2
server {
        listen 80;
        server_name {{ wordpress_server_name }};
        root {{ wordpress_root_path }};
        client_max_body_size 100m;

        location / {
                index index.php;
        }

        location ~\.php$ {
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param HTTPS on;
                include fastcgi_params;
        }
}


# 添加变量
[root@ansible roles]# vim group_vars/all 
# wordpress
wordpress_server_name: blog.hualaotou.com
wordpress_root_path: /code/wordpress

# 上传压缩包
[root@ansible files]# pwd
/opt/roles/wordpress/files
[root@ansible files]# ll
总用量 16868
-rw-r--r--. 1 root root 17270810 11月 27 14:31 wordpress-5.8.2-zh_CN.zip


# 添加触发器
[root@ansible roles]# vim wordpress/handlers/main.yml
- name: Restart Nginx server
  service: name=nginx state=restarted
  
# 添加play
[root@ansible roles]# vim site.yml
- hosts: webserver
  roles:
    - role: wordpress
#    - role: nginx
#    - role: php-fpm   wordpress中有meta，添加了依赖
  tags: web

# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml -t web


```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211127153702.png)



##### WordPress_proxy引入

```shell
[root@ansible roles]# mkdir wordpress_proxy/{tasks,handlers,templates,files，meta} -p
# 添加依赖
[root@ansible roles]# vim wordpress_proxy/meta/main.yml
dependencies:
- role: nginx
- role: keepalived

[root@ansible roles]# vim wordpress_proxy/tasks/main.yml
- name: Create  SSL Directory
  file: path=/etc/nginx/ssl_key state=directory  owner={{ web_user }} group={{ web_user }}

- name: Copy SSL public Pri Key  
  copy: src={{ item }}  dest=/etc/nginx/ssl_key
  with_items:
    - server.crt
    - server.key

- name: Wordpress Proxy Virt
  template: src=wordpress_proxy.conf.j2  dest=/etc/nginx/conf.d/wordpress_proxy.conf
  notify: Restart Nginx Server


# 准备证书
[root@ansible files]# ll
总用量 0
-rw-r--r--. 1 root root 0 11月 27 17:19 server.crt
-rw-r--r--. 1 root root 0 11月 27 17:19 server.key
[root@ansible files]# pwd
/opt/roles/wordpress_proxy/files

# 触发器
[root@ansible roles]# vim wordpress_proxy/handlers/main.yml
- name: Restart Nginx Server
  service: name=nginx  state=restarted

# 配置模板
[root@ansible roles]# vim wordpress_proxy/templates/wordpress_proxy.conf.j2 

upstream {{ wordpress_server_name }} {
  {% for i in group['webserver']%}
        server {{ i }};
  {% endfor %}

}

server {
        listen 80;
        server_name {{ wordpress_server_name }};
        return 302 https://$server_name$request_uri;

}
server {
        listen 443 ssl;
        server_name {{ wordpress_server_name }};
        ssl_certificate ssl_key/server.crt;
        ssl_certificate_key ssl_key/server.key;

        location / {
                proxy_pass http://{{ wordpress_server_name }};
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                include proxy_params;
        }

}


# 添加play
[root@ansible roles]# vim site.yml
- hosts: lbserver
  roles:
    - role: wordpress_proxy
#    - role: nginx
#    - role: keepalived
  tags: lb

# 执行playbook
[root@ansible roles]# ansible-playbook -i hosts site.yml -t lb

```





