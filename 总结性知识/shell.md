# shell

### 基础概念

* 快捷键

```shell
1. 命令和文件自动补齐
2. 命令历史记忆功能	上下键、!number(命令的序号)、!$
3. 快捷键	
	Ctrl+a 切换到命令行开始(跟home一样，但是home在某些unix环境下无法使用)
	Ctrl+e 切换到命令行末尾
	Ctrl+u 清除剪切光标之前的内容
	Ctrl+k 清除剪切光标之后的内容
	ctrl+y	粘贴刚才锁删除的字符
	ctrl+左右键  快速移动光标
	Ctrl+r	在历史命令中查找，输入关键字调出之前的命令
4. 前后台作业控制  &、nohup、^C、^Z(停止),bg %1(将一个在后台暂停的命令，变成继续执行)、fg (将后台中的命令调至前台继续运行)%1、kill %3、screen
5. 输入输出重定向 0,1,2  > >> 2> 2>> 2>&1 &>        cat < /etc/hosts         cat <<EOF         cat >file1 <<EOF
6. 管道	| tee
7. 命令排序
		
		&&    ||		;   具备逻辑判断
	    ;   无论前面是否执行成功，分号后的命令都会继续执行	
		&&:前面执行成功，后面的才继续执行
		||：前面命令不成功，后面的命令也会继续
		./configure && make && make install	（命令返回值 echo $?）
	注意：
		command &							后台执行
		command &>/dev/null			混合重定向（标准输出1，错误输出2）
		command1 && command2		命令排序，逻辑判断
8. shell通配符（元字符）表示的不是本意
		*	匹配任意多个字符  ls in*   rm -rf *  rm -rf *.pdf  find / -iname "*-eth0" yum -y install epel*
		?	匹配任意一个字符  touch love loove live l7ve; ll l?ve
		[]	匹配括号中任意一个字符 [abc] [a-z] [0-9] [a-zA-Z0-9]
		()	在子shell中执行(cd /boot;ls)   (umask 077; touch file1000)    
		注意：如何查看当前子shell     ps | grep $$    $$:当前shell的pid
		{} 集合	touch file{1..9} 
		
	\	转义符，让元字符回归本意
	
```

### 脚本格式

```shell
 #!/bin/bash                #!/usr/bin/env bash
	    指定命令解释器:第一行的专门解释命令解释器
	    注释 ：以#开有的都不生效
	    编写bash指令集合
	    
#!/bin/bash
#注释
#author：blackmed
#version：0.1
#功能

bash脚本的执行:
#./scripts
#/shelldoc/scripts
#. ./scripts               使用当前登录shell执行
#source ./scripts      使用当前登录shell执行  比如cd /tmp会改变当前shell环境，但是其他的方式不会
#bash scripts

bash脚本调试
	•sh –x script
		这将执行该脚本并显示所有变量的值
	•sh –n script
		不执行脚本只是检查语法模式，将返回所有错误语法
	•sh –v script
		执行脚本前把脚本内容显示在屏幕上
		
位置变量
		$1 $2 $3 $4 $5 $6 $7 $8 $9 ${10}
预定义变量
	    $0    脚本名
		$*		所有的参数
		$@ 	所有的参数
		$# 	参数的个数
		$$ 	当前进程的PID
		$!     上一个后台进程的PID
		$?		上一个命令的返回值 0表示成功	

```

#### 赋值方法

```shell
1. 显式赋值
		变量名=变量值
		示例：
		ip1=192.168.1.251
		school="BeiJing 1000phone"
		today1=`date +%F`
		today2=$(date +%F)
2. read 从键盘读入变量值
		read 变量名
		read -p "提示信息: "  变量名
		read -t 5 -p "提示信息: "  变量名
		• -t 后面跟秒数，定义输入字符的等待时间
		read -n 2 变量名
		• -n 后跟一个数字，定义输入文本的长度，很实用。
		
示例3：
		# vim first.sh
		back_dir1=/var/backup
		read -p "请输入你的备份目录: " back_dir2
		echo $back_dir1
		echo $back_dir2
		# sh first.sh
		
定义或引用变量时注意事项：
		" "  	弱引用 可以实现变量和命令的替换
		' ' 	    强引用 不完成变量替换
```

#### 变量运算

```shell
1. 整数运算
		方法一：expr
        expr 1 + 2
		expr $num1 + $num2    			+  -  \*  /  %
		
		方法二：$(())
		echo $(($num1+$num2))      	+  -  *  /   %
		echo $((num1+num2))
		echo $((5-3*2))	 
		echo $(((5-3)*2))
		echo $((2**3))
		sum=$((1+2)); echo $sum

		方法三：$[]
		echo $[5+2]						         +  -  *  /  %
		echo $[5**2]

		方法四：let
		let sum=2+3; echo $sum
		let i++; echo $i
		
```



#### 变量内容删除/替换

```shell
==="内容"的删除===
[root@xingdian ~]# url=www.sina.com.cn
[root@xingdian ~]# echo ${#url}			获取变量值的长度
15
[root@xingdian ~]# echo ${url}			    标准查看
www.sina.com.cn
[root@xingdian ~]# echo ${url#*.}		从前往后，最短匹配
sina.com.cn
[root@xingdian ~]# echo ${url##*.}		从前往后，最长匹配	贪婪匹配
cn

[root@xingdian ~]# url=www.sina.com.cn
[root@xingdian ~]# echo ${url}
www.sina.com.cn
[root@xingdian ~]# echo ${url%.*}		从后往前，最短匹配
www.sina.com
[root@xingdian ~]# echo ${url%%.*}	从后往前，最长匹配	贪婪匹配
www

[root@xingdian ~]# url=www.sina.com.cn
[root@xingdian ~]# echo ${url#a.}
www.sina.com.cn
[root@xingdian ~]# echo ${url#*sina.}
com.cn

[root@xingdian ~]# echo $HOSTNAME
xingdian.1000phone.com
[root@xingdian ~]# echo ${HOSTNAME%%.*}
xingdian

索引及切片
[root@xingdian ~]# echo ${url:0:5}
0:从头开始
5:到第五个
[root@xingdian ~]# echo ${url:5:5}		
[root@xingdian ~]# echo ${url:5}

==="内容"的替换===
[root@xingdian ~]# url=www.sina.com.cn
[root@xingdian ~]# echo ${url/sina/baidu}
www.baidu.com.cn

[root@xingdian ~]# url=www.sina.com.cn
[root@xingdian ~]# echo ${url/n/N}
www.siNa.com.cn
[root@xingdian ~]# echo ${url//n/N}		贪婪匹配
www.siNa.com.cN


===变量的替代===
[root@xingdian ~]# unset var1
[root@xingdian ~]# 
[root@xingdian ~]# echo ${var1}
[root@xingdian ~]# echo ${var1-aaaaa}
aaaaa

[root@xingdian ~]# var2=111
[root@xingdian ~]# echo ${var2-bbbbb}
111
[root@xingdian ~]# 
[root@xingdian ~]# var3=
[root@xingdian ~]# echo ${var3-ccccc}

${变量名-新的变量值}
变量没有被赋值：会使用“新的变量值“ 替代

[root@xingdian ~]# unset var1
[root@xingdian ~]# unset var2
[root@xingdian ~]# unset var3
[root@xingdian ~]# 
[root@xingdian ~]# var2=
[root@xingdian ~]# var3=111
[root@xingdian ~]# echo ${var1:-aaaa}
aaaa
[root@xingdian ~]# echo ${var2:-aaaa}
aaaa
[root@xingdian ~]# echo ${var3:-aaaa}
111

${变量名:-新的变量值}
变量没有被赋值（包括空值）：都会使用“新的变量值“ 替代
变量有被赋值： 不会被替代

[root@xingdian ~]# echo ${var3+aaaa}
[root@xingdian ~]# echo ${var3:+aaaa}

[root@xingdian ~]# echo ${var3=aaaa}
[root@xingdian ~]# echo ${var3:=aaaa}

[root@xingdian ~]# echo ${var3?aaaa}
[root@xingdian ~]# echo ${var3:?aaaa}

自增运算
i++   ++i
[root@xingdian ~]# let x=i++         先赋值，再运算
[root@xingdian ~]# let y=++j         先运算，再赋值
```

### 流程控制

#### 条件测试

```shell
格式1： test 条件表达式
格式2： [ 条件表达式 ]
格式3： [[ 条件表达式 ]]

1、文件测试
[ -d dir ] 		是否是目录
[ -f file ]		是否存在，而且是文件
[ -r file ]		当前用户对该文件是否有读权限
[ -x file ]		当前用户对该文件是否有执行权限
[ -w file ]		当前用户对改文件是否有写权限

2、数值比较 [ 整数1 操作符 整数2 ]
[ 1 -gt 10 ]	大于
[ 1 -lt 10 ]	    小于
[ 1 -eq 10 ]	等于
[ 1 -ne 10 ]  	不等于
[ 1 -ge 10 ]	大于等于
[ 1 -le 10 ]	小于等于

3、字符串比较
提示：使用双引号
[root@xingdian ~]# [ "$USER" = "root" ];echo $?
0
[root@xingdian ~]# [ "$USER" == "root" ];echo $?
0
注意：
""：弱引用，可以实现变量和命令的替换
```

#### 条件判断

```shell
条件判断
If代码返回0表示真，非0为假

==if 语句
    = 单分支结构

        if 条件测试
            then 命令序列
        fi

    = 双分支结构
        if 条件测试
            then 命令序列
            else 命令序列
        fi

    =多分支结构
        if 条件测试1
      	 	 then 命令序列

        [elif 条件测试2
      		 then 命令序列

        elif 条件测试3 
       		 then 命令序列]...

        else 命令序列
        fi

示例：
if [ "$USER" = "root" ]
    then
            if [ $UID -eq 0 ]
                then
                        echo "the user is root"
           fi             
            
    elif
            echo "……"
    elif
            echo "……"
    else
            echo "the user is not root"
            echo "正在给用户授权"
            
fi    
```

### 循环结构

#### for循环

```shell
for 变量名 [ in 取值列表 ]
do
	循环体
done

案例1: ping测试主机
#!/bin/bash
for i in {70..100}
do
        ping -c1 10.30.161.$i &> /dev/null
        if [ $? -eq 0 ]
        then
                echo "10.30.161.$i is up" |tee -a ipup.txt
        else
                echo "10.30.161.$i is down" |tee -a ipdown.txt
        fi

done
```

#### while循环

```shell
一、while语句结构
    while 条件测试
    do
        循环体
    done
    当条件测试成立（条件测试为真），执行循环体

二、until语法结构
    until 条件测试
    do
        循环体
    done
    当条件测试成立（条件测试为假），执行循环体
    
while循环案例：
#!/usr/bin/env bash 
#while 循环练习
num=1
while [ $num -le 3 ]
    do
        read -p "请输入一个密码：" pass
        long_pass=${#pass}
        if [ $long_pass -ge 6 ] && [ $long_pass -le 10 ];then
            echo "密码输入合格"
            break
        else
            echo "密码输入不合格"
        fi
            let num++
    done
```

#### case模式匹配

```shell
一、case 语法结构
read -p "请输入你的选项：" num
case num in
1)                     选项  
	命令序列1   命令/if语句/for循环……
	;;
模式2)
	命令序列2
	;;
模式3)
	命令序列3
	;;
*)
	无匹配后命令序列
esac

示例：
#!/bin/bash
while :
do
cat <<-eof
a.关闭防火墙
b.安装nginx
c.nginx访问量统计PV
d.创建10个用户
e.退出
eof
read -p "请输入你的选择：" num
case $num in 
"a")
   systemctl stop firewalld && systemctl disable firewalld
   ;;

"b")
   yum -y install nginx
   ;;
"c")
   cat /var/log/nginx/access.log |  awk '{print $1}' | wc -l
   ;;
"d")
   for i in {1..10}
   do
        useradd dian$i && echo "123" |passwd --stdin dian$i &>/dev/null
        if [ $? -eq 0 ]
        then
                echo "dian$i is created"
        else
                echo "dian$i is not created"
        fi
   done
   ;;
"e")
   exit
   ;;
esac
done
```

####  循环控制

```shell
break 循环控制：如果有正确的输入就中断循环
sleep 10           //等待10秒，再继续下一操作  
continue:跳出当前这一次循环
exit 直接退出
wait    //wait后没有指定进程号,所以要等待所有进程执行结束后才退出,这里是等待10秒后退出
使用wait等待所有子任务结束
```

### shell函数和正则

#### shell 函数

```shell 
myfunc()		//函数定义
{
echo “This is my first shell function”
}
myfunc			//函数调用
unset myfunc	//取消函数
```

#### 数组

```shell
Shell数组变量

普通数组：只能使用整数作为数组索引
关联数组：可以使用字符串作为数组索引   

name=xingdian    变量 切片
x   i   n   g   d   i   a   n
0   1   2   3   4   5   6   7  索引

books=(linux shell awk docker k8s) 普通数组
linux   shell   awk     docker      k8s
0           1        2          3              4   索引

info=([name]=xingdian [sex]=male [height]=185)  关联数组
xingdian    male    185
name          sex     height  索引

${数组名[索引]}
# declare -a   定义为普通数组
declare -a array1='([0]="pear" [1]="apple" [2]="orange" [3]="peach")'
declare -a array2='([0]="tom" [1]="jack" [2]="alice")'

访问数组元数：
# echo ${array1[0]}			访问数组中的第一个元数
# echo ${array1[@]}			访问数组中所有元数  等同于 echo ${array1[*]}
# echo ${#array1[@]}		统计数组元数的个数
# echo ${!array2[@]}		获取数组元数的索引
# echo ${array1[@]:1}		从数组下标1开始
# echo ${array1[@]:1:2}	从数组下标1开始，访问两个元素

遍历数组：
方法一： 通过数组元数的个数进行遍历
方法二： 通过数组元数的索引进行遍历

访问数组元数：
# echo ${ass_array2[index2]}			访问数组中的第二个元数
# echo ${ass_array2[@]}					访问数组中所有元数  等同于 echo ${array1[*]}
# echo ${#ass_array2[@]}				获得数组元数的个数
# echo ${!ass_array2[@]}				    获得数组元数的索引
```

#### 正则

```shell
匹配数字:     ^[0-9]+$                                       123 456 +表示前面的内容出现多次
匹配Mail： 	[a-z0-9_]+@[a-z0-9]+\.[a-z]+	     yangsheng131420@126.com
匹配IP：		[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}           {1,3}数字出现1-3次


二、元字符
定义：元字符是这样一类字符，它们表达的是不同于字面本身的含义

vim示例：
:1,$ s/tom/David/g		       //如tom、anatomy、tomatoes及tomorrow中的“tom”被替换了，而Tom确没被替换
:1,$ s/\<[Tt]om\>/David/g


1. 正则表达式元字符：
===基本正则表达式元字符
元字符			功能										    示例	
========================================================	
^					行首定位符							    ^love	
$					行尾定位符							    love$		
.					匹配单个字符							l..e		
*					匹配前导符0到多次				    ab*love
.*                 任意多个字符
[]					匹配指定范围内的一个字符		[lL]ove
[ - ]				匹配指定范围内的一个字符		[a-z0-9]ove
[^]				匹配不在指定组内的字符			[^a-z0-9]ove
\					用来转义元字符						    love\.	
\<			        词首定位符							    \<love						
\>				    词尾定位符							    love\>						
\(..\)				匹配稍后使用的字符的标签	    
							:% s/172.16.130.1/172.16.130.5/
							:% s/\(172.16.130.\)1/\15/
							:% s/\(172.\)\(16.\)\(130.\)1/\1\2\35/
							:3,9 s/\(.*\)/#\1/	
																	
x\{m\}			字符x重复出现m次					    o\{5\}
x\{m,\}		    字符x重复出现m次以上			    o\{5,\}						
x\{m,n\}		字符x重复出现m到n次				o\{5,10\}					

===扩展正则表达式元字符
+					匹配一个或多个前导字符			[a-z]+ove	
?					匹配零个或一个前导字符			lo?ve	
a|b				匹配a或b								    love|hate
()					组字符									    loveable|rs    love(able|rs)  ov+ (ov)+
(..)(..)\1\2	    标签匹配字符						    (love)able\1er
x{m}			    字符x重复m次							o{5}		
x{m,}			字符x重复至少m次				        o{5,}
x{m,n}		    字符x重复m到n次					    o{5,10}


空行
/^$/
注释行
/^#/


```

###  三剑客

####  sed

```shell
sed 是一种在线的、非交互式的编辑器，它一次处理一行内容。

语法：
   sed [options]  ‘command’ in_file[s]
options 部分：
    
	-n   静默输出(不打印默认输出) sed -n '1p' a.txt   想显示第几行就显示第几行	
	-e  给予sed多个命令的时候需要-e选项
		#sed -e 's/root/haha/g' -e 's/bash/wwwww/g' passwd > passwd.bak		
		如果不用-e选项也可以用分号“;”把多个命令隔开。
		#sed 's/haha/ro/g ; s/wwwww/kkkk/g' passwd | less	这个是-e的结果
	-i  -i后面没有扩展名的话直接修改文件，如果有扩展名备份源文件，产生以扩展名结尾的新文件
	    	#sed -iback1 -e 's/root/rottt/g' -e 's/bash/wwwww/g' passwd      //选项-i后面没有空格	
		[root@localhost 桌面]# ls
		manifest.txt  passwdback1 	
	-f  当有多个要编辑的项目时，可以将编辑命令放进一个脚本里，再使用sed搭配-f选项
		[root@localhost 桌面]# cat s.sed
		s/bin/a/g
		s/ftp/b/g
		s/mail/c/g
		[root@localhost 桌面]# sed -f s.sed passwd | less
		
		
	正则表达式：
	        基本正则  sed    （遇到扩展正则，记得将符号转义）
	        扩展正则  sed -r   无论是扩展正则还是基本正则，全部加r参数
		
	p   打印行 1p 输出再打印一遍第一行 1~2 打印奇数 0~2打印偶数	
    d  删除文本
        #sed '1 d' passwd
        #sed '$ d' passwd
        #sed '1,3 d' passwd
        #sed '1,/^dian/  d' passwd
    a  追加文本(后)
        #sed '2 a nihao' passwd
        #sed '/^dian/ a nihao' passwd
    i    前插
        # sed -i '1 i nihao' passwd
    c   替换 sed '/zhong/c abc' 将zhong这一行替换成abc
        #sed -i '1 c no' passwd
    r  可使用扩展正则
    
sed流编辑器用法及解析
sed: stream editor(流编辑器)的缩写. 它们最常见的用法是进行文本的替换. 
1. sed可以从stdin中读取内容
$ cat filename | sed 's/pattern/replace_string/'

2. 选项-i会使得sed用修改后的数据替换原文件
$ sed -i 's/pattern/replace_string/' filename

3. g标记可以使sed执行全局替换
$ sed 's/pattern/replace_string/g' filename

4. g标记可以使sed匹配第N次以后的字符被替换
$ echo "thisthisthisthis" | sed 's/this/THIS/2g'

5. sed中的分隔符可以替换成别的字符, 因为s标识会认为后面的字符为分隔符
$ sed 's:text:replace_text:'
$ sed 's|text|replace_text|'

6. sed可以利用指令来删除文件中的空行
$ sed '/^$/d' filename

7. 替换指定的字符串或数字
$ cat sed_data.txt
11 abc 111 this 9 file contains 111 11 99 numbers 0000
$ sed -i 's/\b[0-9]\{3\}\b/NUMBER/g' sed_data.txt
$ cat sed_data.txt
11 abc NUMBER this 9 file contains NUMBER 11 99 numbers 0000

8. 由于在使用-i参数时比较危险, 所以我们在使用i参数时在后面加上.bak就会产生一个备份的文件,以防后悔
$ sed -i.bak 's/pattern/replace_string/' filename
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211109101430.png)

**替换tcp2.conf文件中的部分内容**

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211109102011.png)

#### awk

```shell
一、awk简介
        awk 是一种编程语言，用于在linux/unix下对文本和数据进行处理。
awk -F":" '{print $1}' 文件
awk BEGIN{} {} END{} 文件
二、awk的语法格式
awk [options] 'commands' filenames
＝＝options：
-F   定义输入字段分隔符，默认的分隔符是空格或制表符(tab)
＝＝command：
BEGIN{}       {}         	    END{}
行处理前      行处理 		行处理后
awk 选项 '[BEGIN{}] {} [END{}]' 文件

	BEGIN{} 所有文本内容读入之前要执行的命令  可以不需要后面跟文件，因为他是在读入文件之前的操作
	{}:主输入循环 读入一行命令执行一次循环
	END{}:所有文本都读入完成之后执行的命令    必须要读入文件，因为他是在读入文件之后的操作

＝＝记录与字段相关内部变量：man awk
$0：	    awk变量$0保存当前记录的内容				                
    # awk -F: '{print $0}' /etc/passwd
    [root@hualaotou02 ~]# awk -F: '{print $0}' /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
	
NR：	  打印行号  	
    # awk -F: '{print NR, $0}' /etc/passwd 
    [root@hualaotou02 ~]# awk -F: '{print NR, $0}' /etc/passwd /etc/hosts
    1 root:x:0:0:root:/root:/bin/bash
    2 bin:x:1:1:bin:/bin:/sbin/nologin
    3 daemon:x:2:2:daemon:/sbin:/sbin/nologin
FNR：  打印行号     
    # awk -F: '{print FNR, $0}' /etc/passwd /etc/hosts
    [root@hualaotou02 ~]# awk -F: '{print FNR, $0}' /etc/passwd
    1 root:x:0:0:root:/root:/bin/bash
    2 bin:x:1:1:bin:/bin:/sbin/nologin

NF：	    打印字段、保存记录的字段数，$1,$2...$100			 
    # awk -F: '{print $0,NF}' /etc/passwd
    [root@hualaotou02 ~]# awk -F: '{print $0,NF}' /etc/passwd
    root:x:0:0:root:/root:/bin/bash 7
    bin:x:1:1:bin:/bin:/sbin/nologin 7

FS：	    输入字段分隔符，默认空格					 

    # awk -F'[ :\t]' '{print $1,$2,$3}' /etc/passwd	
    [root@hualaotou02 ~]# awk -F'[ :\t]' '{print $1,$2,$3}' /etc/passwd
    root x 0
    bin x 1

    #awk 'BEGIN{FS=":"} {print $1,$3}' /etc/passwd
    [root@hualaotou02 ~]# awk 'BEGIN{FS=":"} {print $1,$3}' /etc/passwd
    root 0
    bin 1

OFS：   输出字段分隔符				 

    # awk 'BEGIN{FS=":"; OFS="+++"} /^root/{print $1,$2,$3,$4}' /etc/passwd
    [root@hualaotou02 ~]# awk 'BEGIN{FS=":"; OFS="+++"} /^root/{print $1,$2,$3,$4}' /etc/passwd
    root+++x+++0+++0
    [root@hualaotou02 ~]# awk 'BEGIN{FS=":"; OFS="+++"} {print $1,$2,$3,$4}' /etc/passwd
    root+++x+++0+++0
    bin+++x+++1+++1
    daemon+++x+++2+++2

RS        换行符
	# awk -F: 'BEGIN{RS=" "} {print $0}' a.txt
ORS      换行符 
    # awk -F: 'BEGIN{ORS=""} {print $0}' passwd
    [root@hualaotou02 ~]# awk -F: 'BEGIN{ORS=""} {print $0}' /etc/passwd
    root:x:0:0:root:/root:/bin/bashbin:x:1:1:bin:/bin:/sbin/nologindaemon:x:2:2:daemon:/sbin:/sbin/nologinadm:x:3:4:adm:/var/adm:/sbin/nologinlp:x:4:7:lp:/var/spool/lpd:/sbin/nologinsync:x:5:0:sync:/sbin:/bin/syncshutdown:x:6:0:shutdown:/sbin:/sbin/shutdownhalt:x:7:0:

区别：
字段分隔符: FS OFS  默认空格或制表符
记录分隔符: RS ORS 默认换行符

＝＝格式化输出：
print函数
# date |awk '{print "Month: " $2 "\nYear: " $NF}'
# awk -F: '{print "username is: " $1 "\t uid is: " $3}' /etc/passwd
# awk -F: '{print "\tusername and uid: " $1,$3 "!"}' /etc/passwd

printf函数
# awk -F: '{printf "%-15s %-10s %-15s\n", $1,$2,$3}'  /etc/passwd
[root@hualaotou02 ~]#  awk -F: '{printf "%-15s %-10s %-15s\n", $1,$2,$3}'  /etc/passwd
root            x          0              
bin             x          1              
daemon          x          2
# awk -F: '{printf "|%-15s| %-10s| %-15s|\n", $1,$2,$3}' /etc/passwd
[root@hualaotou02 ~]# awk -F: '{printf "|%-15s| %-10s| %-15s|\n", $1,$2,$3}' /etc/passwd
|root           | x         | 0              |
|bin            | x         | 1              |
|daemon         | x         | 2              |
|adm            | x         | 3              |
|lp             | x         | 4              |
 
%s 字符类型
%d 数值类型
%f 浮点类型
占15字符
- 表示左对齐，默认是右对齐
printf默认不会在行尾自动换行，加\n
========================================================


三、awk模式和动作
	任何awk语句都由模式和动作组成。模式部分决定动作语句何时触发及触发事件。处理即对数据进行的操作。
如果省略模式部分，动作将时刻保持执行状态。模式可以是任何条件语句或复合语句或正则表达式。模式包括两
个特殊字段 BEGIN和END。使用BEGIN语句设置计数和打印头。BEGIN语句使用在任何文本浏览动作之前，之
后文本浏览动作依据输入文本开始执行。END语句用来在awk完成文本浏览动作后打印输出文本总数和结尾状态。

模式可以是：

==正则表达式：
匹配记录（整行）：～匹配
# awk '/^alice/'  /etc/passwd
# awk '$0 ~ /^alice/'  /etc/passwd
# awk '!/alice/' passwd
# awk '$0 !~ /^alice/'  /etc/passwd

匹配字段：匹配操作符（~ !~）
# awk -F: '$1 ~ /^alice/'  /etc/passwd
# awk -F: '$NF !~ /bash$/'  /etc/passwd

  
==比较表达式：
比较表达式采用对文本进行比较，只有当条件为真，才执行指定的动作。比较表达式使用关系运算符，
用于比较数字与字符串。

关系运算符
运算符			含义						    示例
<					小于						    x<y
<=				小于或等于			       x<=y
==				等于						    x==y
!=				    不等于					    x!=y
>=				大于等于				    x>=y
>					大于						    x>y

# awk -F: '$3 == 0' /etc/passwd
# awk -F: '$3 < 10' /etc/passwd
# awk -F: '$NF == "/bin/bash"' /etc/passwd
# awk -F: '$1 == "alice"' /etc/passwd
# awk -F: '$1 ~ /alic/ ' /etc/passwd
# awk -F: '$1 !~ /alic/ ' /etc/passwd
# df -P | grep  '/' |awk '$4 > 25000'


==条件表达式：
# awk -F: '$3>300 {print $0}' /etc/passwd
# awk -F: '{ if($3>300) {print $0} }' /etc/passwd
# awk -F: '{ if($3>300) {print $3} else{print $1} }' /etc/passwd


==算术运算：+ - * / %(模) ^(幂2^3)
可以在模式中执行计算，awk都将按浮点数方式执行算术运算
# awk -F: '$3 * 10 > 500' /etc/passwd
# awk -F: '{ if($3*10>500){print $0} }' /etc/passwd


＝＝逻辑操作符和复合模式
&&		逻辑与		a&&b
||			逻辑或		a||b
!			逻辑非		!a   除了这个以外的
# awk -F: '$1~/root/ && $3<=15' /etc/passwd
# awk -F: '$1~/root/ || $3<=15' /etc/passwd
# awk -F: '!($1~/root/ || $3<=15)' /etc/passwd

四、awk脚本编程
＝＝条件判断
if语句：
格式
{if(表达式)｛语句;语句;...｝}

awk -F: '{if($3==0) {print $1 " is administrator."}}' /etc/passwd
awk -F: '{if($3>0 && $3<1000){count++;}}  END{print count}' /etc/passwd	　　//统计系统用户数

if...else语句:
格式
{if(表达式)｛语句;语句;...｝else{语句;语句;...}}
awk -F: '{if($3==0){print $1} else {print $7}}' /etc/passwd
awk -F: '{if($3==0) {count++} else{i++} }' /etc/passwd
awk -F: '{if($3==0){count++} else{i++}} END{print "管理员个数: "count ; print "系统用户数: "i}' /etc/passwd

if...else if...else语句：
格式
{if(表达式1)｛语句;语句；...｝else if(表达式2)｛语句;语句；...｝else if(表达式3)｛语句;语句；...｝else｛语句;语句；...｝}
awk -F: '{if($3==0){i++} else if($3>999){k++} else{j++}} END{print i; print k; print j}' /etc/passwd
awk -F: '{if($3==0){i++} else if($3>999){k++} else{j++}} END{print "管理员个数: "i; print "普通用个数: "k; print "系统用户: "j}' /etc/passwd 


＝＝循环
$1 ~ /^ll/
while:
[root@xingdian ~]# awk 'BEGIN{ i=1; while(i<=10){print i; i++}  }'
[root@xingdian ~]# awk -F: '/^root/{i=1; while(i<=7){print $i; i++}}' passwd
[root@xingdian ~]# awk  '{i=1; while(i<=NF){print $i; i++}}' /etc/hosts
[root@xingdian ~]# awk -F: '{i=1; while(i<=10) {print $0;  i++}}' /etc/passwd	   //将每行打印10次

＝＝数组(统计)
# awk -F: '{username[++i]=$1} END{print username[1]}' /etc/passwd
root
# awk -F: '{username[i++]=$1} END{print username[1]}' /etc/passwd 
bin
# awk -F: '{username[i++]=$1} END{print username[0]}' /etc/passwd 
root

数组遍历：
1. 按索引遍历
2. 按元素个数遍历

==按元素个数遍历==
# awk -F: '{username[x++]=$1} END{for(i=0;i<x;i++) print i,username[i]}' /etc/passwd
# awk -F: '{username[++x]=$1} END{for(i=1;i<=x;i++) print i,username[i]}' /etc/passwd


==按索引遍历==
# awk -F: '{username[x++]=$1} END{for(i in username) {print i,username[i]} }' /etc/passwd
# awk -F: '{username[++x]=$1} END{for(i in username) {print i,username[i]} }' /etc/passwd
注：变量i是索引

1. 统计/etc/passwd中各种类型shell的数量(统计谁把谁当作索引)
[root@xingdian ~]# awk -F: '{shells[$NF]++} E,\"$var\")"
bash scriptND{ for(i in shells){print i,shells[i]} }' /etc/passwd
```

![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20211112160601.png)

#### expect

```shell
yum -y install expect
/usr/bin/expect <<eof
spawn:生成 spawn ssh 10.18.44.196
expect:捕获 expect "password"
send:发送  send "1\n"
expect eof
eof
```

#### grep

```shell
grep:  	在文件中全局查找指定的正则表达式，并打印所有包含该表达式的行
egrep: 	扩展的egrep，支持更多的正则表达式元字符
fgrep: 	固定grep(fixed grep)，有时也被称作快速(fast grep)，它按字面解释所有的字符(了解)

一、grep命令格式
grep [选项] PATTERN filename filename ...
# grep 'Tom' /etc/passwd
# grep 'bash shell' /etc/test	
找到：				        grep返回的退出状态为0
没找到：				    grep返回的退出状态为1
找不到指定文件：	    grep返回的退出状态为2

二、grep使用的元字符
grep:						使用基本元字符集	^, $, ., *, [], [^], \< \>,\(\),\{\}, \+, \|
egrep(或grep -E):	使用扩展元字符集	?, +, { }, |, ( )
注：grep也可以使用扩展集中的元字符，仅需要对这些元字符前置一个反斜线

\w					        所有字母与数字，称为字符[a-zA-Z0-9]		 'l[a-zA-Z0-9]*ve'	        'l\w*ve'
\W				        所有字母与数字之外的字符，称为非字符	'love[^a-zA-Z0-9]+' 	    'love\W+'
\b					        词边界						     '\<love\>'		          '\blove\b'		

三、grep选项
-i, --ignore-case				忽略大小写
-l, --files-with-matches		只列出匹配行所在的文件名
-n, --line-number				在每一行前面加上它在文件中的相对行号
-c, --count							显示成功匹配的行数
-s, --no-messages				禁止显示文件不存在或文件不可读的错误信息
-q, --quiet, --silent				静默--quiet, --silent
-v, --invert-match				反向查找，只显示不匹配的行
-R, -r, --recursive				递归针对目录
--color								颜色
-o, --only-matching			只显示匹配的内容

```

