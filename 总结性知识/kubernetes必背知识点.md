# kubernetes必背知识点

###  发展

* 发展历史

  ```shell
  # kubernetes    背后是Google     在10年前就已实现容器化基础架构     borg系统   ⇒   采用go语言对borg系统进行翻写
  
  # k8s 是一个全新的基于容器技术的分布式架构领先方案，它不局限于任何一种语言，没有限定任何编程接口，所以任何编程语言的服务，都可以被映射到kubernetes的service，并通过标准的TCP通信协议进行交互；
  
  # k8s 的优点：
  	# 是一个完备的分布式系统支撑平台；
  	# 具有而完备的集群管理能力（安全防护、准入机制、智能负载均衡、滚动升级、自愈、扩容···）
  	# 提供了完善的管理工具：开发、部署测试、运维监控···
  	
  # Service :分布式集群架构的核心
  	# 拥有唯一指定的名称
  	# 拥有一个虚拟的IP和端口号
  	# 能够提供某种远程服务能力
  	# 被映射到提供这种服务能力的一组容器应用上
  ```

* 互联网的飞跃
  * 第一次： C/S（服务器/客户端）--> B/S(浏览器操作)
  * 第二次： 移动互联网的飞跃，只需要一个客户端自己可以操作
  * 第三次： 小程序的诞生
* 使用docker容器化封装应用程序的缺点
  * 单机使用，无法有效的集群化
  * 随着容器数量的上升，管理成本攀升
  * 没有有效的容灾、自愈机制
  * 没有预设编排模板，无法实现快速、大规模容器调度
  * 没有统一的配置管理中心工具
  * 没有容器声明周期的管理工具
  * 没有图形化运维管理工具

###  基础名词

* 服务状态：

  * 有状态服务：DBMS
  * 无状态服务： LVS APACHE
  * 高可用集群副本数据最好是 >= 3   奇数个

* 基础名词：

  ```shell 
  apiserver: 所有服务访问的统一入口
  crontrollermanager: 维持副本期望数目
  scheduler:	负责介绍任务，选择合适的节点进行分配任务
  etcd： 键值对数据库，  存储k8s集群所有重要的信息（持久化）
  kubelet： 直接跟容器引擎交互实现容器的生命周期管理
  kube-proxy： 负责写入规则至IPtables、ipvs 实现服务映射访问
  CoreDNS： 可以为集群中的svc创建一个域名ip的对应关系解析
  dashboard： 给k8s集群提供一个B|S 结构访问体系（页面管理）
  ingress controller： 官方只能实现四层代理，ingress可以实现七层代理
  federation： 提供一个可以跨集群中心多k8s统一管理功能
  prometheus： 提供k8s集群的监控能力
  ELK： 提供k8s集群日志统一分析介入平台
  
  ```

* 虚拟容器

  ```shell
  PID : 进程编号
  NET : 网络设备、网络协议栈、端口等
  IPC : 信号量、消息队列、共享内存
  MOUNT : 文件系统、挂载点
  UTS : 主机名和主机域
  USER : 操作进程的用户和用户组
  ```

* 四组基本概念

  * pod/pod控制器

  ```shell
  pod： 
  	pod是k8s里面能够被运行的最小逻辑单元（原子单元）
  	1个pod里面可以运行多个容器，他们共享UTS+NET+IPC名称空间
  	可以把pod理解成一个豌豆荚，而同一pod内的每一个容器是一颗颗豌豆
  	一个pod里运行多个容器，又叫：边车（sidecar）模式
  pod控制器：
  	pod控制器是pod启动的一种模板，用来保证在k8s里启动的pod，应始终按照人们的预期运行（副本数、生命周期、健康状态检查）
  	k8s内提供了众多的pod控制器，常用的有以下几种
  		Deployment
  			特点：
    				部署无状态应用，只关心数量，不论角色等，称无状态
  				管理Pod和ReplicaSet
  				具有上线部署、副本设定、滚动升级、回滚等功能
  				提供声明式更新，例如只更新一个新的image
  				应用场景: web 服务
  		Daemonset
  			特点：
  				在每一个Node上运行一个Pod
  				新加入的Node也同样会自动运行一个Pod
  				应用场景：Agent、监控
  		Replicaset
  		Statefulset
  			特点：
  				部署有状态应用
  				解决Pod独立生命周期，保持Pod启动顺序和唯一性
  				稳定，唯一的网络标识符，持久存储(例如: etcd 配置文件，节点地址发生变					化，将无法使用)
  				有序，优雅的部署和扩展、删除和终止(例如: mysql 主从关系，先启动主，再启				动从)
  				有序，滚动更新
  				应用场景: 数据库
  
  		job
  			特点：
  				Job分为普通任务（Job）和定时任务（CronJob）
  				一次性执行
  				应用场景：离线数据处理，视频解码等业务
  		CronJob
  			特点：
  				周期性任务，像Linux的Crontab一样。
  				周期性任务
  				应用场景：通知，备份
  ```

  

  * name/namespace

  ```shell
  name:
  	由于K8S内部，使用“资源”来定义每一种逻辑概念（功能），故每种“资源”，都应该有自己的名称
      “资源”有api版本（apiVersion）、类别（lind）、元数据（metadata）、定义清单（spec）、状态（status）等配置信息
      “名称”通常定义在资源的元数据信息里
      
      
  Namespace：
      随着项目的增多，人员增加、集群规模的扩大，需要一种能够隔离K8S内各种“资源”的方法，这就是名称空间
      名称空间可以理解为K8S内部的虚拟集群组
      不同名称空间内的“资源”，名称可以相同，相同名称空间内的同种“资源”，“名称”不能相同
      合理的使用k8s的名称空间，使得集群管理员能够更好的对交付到K8S里的服务进行分类管理和浏览
      K8S里默认存在的名称空间有：default、kube-system、kube-public
      查询K8S 里特定“资源”要带上相应的名称空间
  ```

  

  * label/label选择器

  ```shell 
  Label：
      标签是K8S特色的管理方式，便于分类管理资源对象
      一个标签可以对应多个资源，一个资源可以有多个标签，他们是多对多的关系
      一个资源拥有多个标签，可以实现不同纬度的管理
      标签的组成    key=value
      与标签类似的，还有一种“注解”（annotation）
      
      
  Label选择器：
      给资源打上标签后，可以使用标签选择器过滤指定的标签
      标签选择器目前有两个：基于等值关系（等于、不等于）和基于集合关系（属于、不属于、存在）
      许多资源支持内嵌标签选择器字段
          matchLabels
          matchExpressions
  ```

  

  * service/ingress

  ```shell
  Service：
      在K8S的世界里，虽然每个Pod都会被分配一个单独的IP地址，但这个IP地址会随着Pod的销毁而消失
      Service（服务）就是用来解决这个问题的核心概念
      一个Service可以看你做一组提供相同服务的Pod的对外访问接口
      Service作用于哪些Pod是通过标签选择器来定义的
      
      
  Ingress：
      Ingress是K8S集群里工作在OSI网络参考模型下，第七层的应用，对外暴露的接口
      Service只能进行L4流量调度，表现形式是IP+ port
      Ingress则可以调度不同业务域、不同URL访问路劲的业务流量
  ```

  

###  核心组件

* 配置存储中心--> etcd服务

* 主控（master）节点：

  * kube-apiserver服务：
    * 提供了集群管理的RESTAPI接口（包括鉴权、数据校验及集群状态变更）
    * 负责其他模块之间的数据交互，陈丹通信枢纽功能
    * 是资源配额控制的入口
    * 提供完备的集群安全机制
  * kube-controller-manager服务：
    * 由一系列控制器组成，通过apiserver监控整个集群的状态，并确保集群处于预期的工作状态
    * Node Controller
    * Deployment Controller
    * Service Controller
    * Volume Controller
    * Namespace Controller
  * kube-scheduler服务：
    * 主要功能是接收调度pod到适合的运算节点上
    * 预算策略  （predict）
    * 优选策略（priorities）

* 运算（node）节点：

  * kube-kubelet服务：
    * 简单地说，kubelet的主要功能就是定时从摸个地方获取节点上pod的期望状态（运行什么容器、运行的副本数量、网络护着存储如何配置等等），并调用对应的容器平台接口达到这个状态
    * 定时汇报当前节点的状态给apiserver，以供调度的时候使用
    * 镜像和容器的清理工作，保证节点上镜像不会占满磁盘空间，退出的容器不会占用太多资源
  * kube-proxy服务：
    * 是K8S在每个节点上运行网络代理，service资源的载体
    * 简历pod网络和集群网络的关系（clusterip→ podip）
    * 常用三种流量调度模式
      * Userspace （废弃）
      * Iptables（濒临废弃）
      * Ipvs（推荐）
    * 负责建立和删除包括更新调度规则、通知apiserver自己的更新、或者从apiserver哪里获取其他kube-proxy的调度规则变化来更新自己的
  * CLI客户端：
    * kubectl
  * 核心插件：
    * CNI网络插件-->flannel/calico
    * 服务发现用插件 → coredns
    * 服务暴露用插件→ traefik
    * GUI管理插件 → Dashboard
  * 网络架构：

  ![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20210906134435.png)

  * 逻辑架构：

  ![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20210906134536.png)

  * 物理架构：

  ![](https://gitee.com/hualaotou/markdone-picture/raw/master/img/20210906134636.png)

##  k8s干货

###  k8s核心资源管理

####  陈述式资源管理

* 查看名称空间

  ```shell
  [root@hdss7-21 ~]# kubectl get namespace
  # 可以将namespace简写成ns
  [root@hdss7-21 ~]# kubectl get ns
  ```

* 查看指定名称空间的所有资源

  ```shell
  [root@hdss7-21 ~]# kubectl get all -n default
  # 查询默认default名称空间时可以不写
  [root@hdss7-21 ~]# kubectl get all
  ```

* 创建名称空间

  ```shell
  [root@hdss7-21 ~]# kubectl create namespace app
  [root@hdss7-21 ~]# kubectl create ns infra
  # 若需要从私有仓库里面获取镜像时，只是login是不够的，需要在名称空间里面创建secret资源
  [root@hdss7-21 ~]# kubectl create secret docker-registry harbor --docker-server=harbor.od.com --docker-username=admin --docker-password=Harbor12345 -n infra
  ```

* 删除名称空间

  ```shell
  [root@hdss7-21 ~]# kubectl delete ns app
  ```

* 创建deployment资源

  ```shell
  [root@hdss7-21 ~]# kubectl create deployment nginx-dp --image=harbor.od.com/public/nginx:v1.7.9 -n kube-public
  ```

* 查看deployment资源

  ```shell
  [root@hdss7-21 ~]# kubectl get deploy -n kube-public
  ```

* 删除deployment

  ```shell
  [root@hdss7-21 ~]# kubectl delete deployment nginx-dp -n kube-public
  ```

  

* 详细显示  -o wide

  ```shell
  [root@hdss7-21 ~]# kubectl get pods -n kube-public -o wide
  [root@hdss7-21 ~]# kubectl describe deployment nginx-dp -n kube-public
  ```

* 查看pod资源

  ```shell
  [root@hdss7-21 ~]# kubectl get pods -n kube-public
  ```

* 实现与pod资源交互

  ```shell
  [root@hdss7-21 ~]# kubectl exec -it nginx-dp-6464d8f7d8-zszvb /bin/bash -n kube-public
  # 也可以实现docker  exec
  ```

* 删除pod资源（相当于重启）

  ```shell
  [root@hdss7-21 ~]# kubectl delete pod nginx-dp-6464d8f7d8-zszvb -n kube-public
  ```

* 实时查看变化过程

  ```shell
  [root@hdss7-22 ~]# watch -n 1 'kubectl describe deployment nginx-dp -n kube-public | grep -C 5 Event' 
  ```

* 创建service

  ```shell
  [root@hdss7-21 ~]# kubectl expose deployment nginx-dp --port=80 -n kube-public
  [root@hdss7-21 ~]# kubectl get all -n kube-public
  ```

* 查看ipvs调度

  ```shell
  [root@hdss7-22 ~]# ipvsadm -Ln
  ```

* 升缩deployment

  ```shell
  [root@hdss7-22 ~]# kubectl scale deployment nginx-dp --replicas=2 -n kube-public
  ```

* 查看service

  ```shell
  [root@hdss7-22 ~]# kubectl get svc -n kube-public（简写）
  [root@hdss7-22 ~]# kubectl get service -n kube-public
  [root@hdss7-22 ~]# kubectl describe svc -n kube-public
  ```

* 删除service

  ```shell
  [root@hdss7-21 ~]# kubectl delete svc nginx-ds
  ```

* 查看node节点

  ```shell
  [root@hdss7-21 conf]# kubectl get node
  ```

* 给node节点添加标签

  ```shell
  [root@hdss7-22 bin]# kubectl label node hdss7-21.host.com node-role.kubernetes.io/node=
  [root@hdss7-22 bin]# kubectl label node hdss7-21.host.com node-role.kubernetes.io/master=
  [root@hdss7-22 bin]# kubectl label node hdss7-22.host.com node-role.kubernetes.io/node=
  [root@hdss7-22 bin]# kubectl label node hdss7-22.host.com node-role.kubernetes.io/master=
  ```

  

* 给node添加标签

####  声明式资源管理

* 声明式资源管理方法依赖于--资源配置清单（yaml/json）

* 查看pod资源配置清单

  ```shell
  [root@hdss7-21 ~]# kubectl get pods nginx-dp-6464d8f7d8-vclsz -o yaml -n kube-public
  ```

* 查看service资源配置清单

  ```shell
  [root@hdss7-21 ~]# kubectl get svc nginx-dp -o yaml -n kube-public
  ```

* 解释资源配置清单

  ```shell
  [root@hdss7-21 ~]# kubectl explain service.metadata
  ```

* 创建资源配置清单

  ```shell
  [root@hdss7-21 ~]# vim /root/nginx-dd-svc.yaml
  apiVersion: v1
  kind: Service 			#表明是kubernetes service
  metadata:
    labels:				#标签
      app: nginx-dd		#service拥有的标签
    name: nginx-dd			#service的全局唯一名称
    namespace: default	
  spec:
    ports:
    - port: 80			#service提供服务的端口号
      protocol: TCP
      targetPort: 80		
    selector:				#service对应的pod拥有这里定义的标签
      app: nginx-dd
    type: ClusterIP
    
  [root@hdss7-21 ~]# kubectl create -f nginx-dd-svc.yaml
  [root@hdss7-21 ~]# kubectl get svc
  ```

* 修改资源配置清单

  ```shell
  离线修改：
  修改nginx-dd-svc.yaml文件，然后使用命令：
  [root@hdss7-21 ~]# kubectl apply -f nginx-dd-svc.yaml
  在线修改：
  直接使用 
  [root@hdss7-21 ~]# kubectl edit service nginx-dd 
  在线编辑资源配置清单，并保存生效
  ```

* 删除资源配置清单

  ```shell
  陈述式删除：
  [root@hdss7-21 ~]# kubectl delete svc nginx-dd 
  
  声明式删除：
  [root@hdss7-21 ~]# kubectl delete -f nginx-dd-svc.yaml
  ```

####  GUI 式资源管理

* dashboard页面点击操作

###  docker相关知识

####  要求

* 内核版本在3.8以上

* daemon.json文件内容解析

  ```shell
  ~]# vi /etc/docker/daemon.json
  {
    "graph": "/data/docker",
    "storage-driver": "overlay2",
    "insecure-registries": ["registry.access.redhat.com","quay.io"],
    "registry-mirrors": ["https://q2gr04ke.mirror.aliyuncs.com"],
    "bip": "172.7.5.1/24",
    "exec-opts": ["native.cgroupdriver=systemd"],
    "live-restore": true
  }
  # 重启docker让配置生效
  ~]# systemctl reset-failed docker.service
  ~]# systemctl start docker.service
  ~]# systemctl restart docker
  
  graph：docker的工作目录，docker会在下面生成一大堆文件
  storage-driver： 存储驱动
  insecure-registries：私有仓库
  registry-mirrors：国内加速源
  bip：docker容器地址
  live-restrore：容器引擎死掉的事情，起来的docker是否继续活着
  ```

####  镜像

* 镜像常规结构

  ```shell
  ${registry_name}/${repository_name}/${image_name}:${tag_name}
  例如：
  docker.io/library/alipine:3.10.1
  harbor.hualaotou.vip/infra/kibana:v6.8.6
  ```

* 登录远程仓库

  ```shell
  ~]# docker login docker.io
  ```

* 镜像常规操作

  ```shell
  拉取镜像：
  ~]# docker pull alpine:3.10.1
  
  重新打tag（按照镜像常规结构）：
  [root@hdss7-200 ~]# docker tag adfab5632ef4 harbor.hualaotou.vip/infra/kibana:v6.8.6
  
  推送镜像：
  [root@hdss7-200 ~]# docker push harbor.hualaotou.vip/infra/kibana:v6.8.6
  ```

* 删除镜像

  ```shell
  ~]# docker rmi 965ea09ff2eb
  ~]# docker rmi -f 965ea09ff2eb
  ```

####  docker高级操作

* 批量删除容器

  ```shell
  # 全部有记录的容器进程
  ~]# docker ps -a
  # 存活的容器进程
  ~]# docker ps
  # 启动容器（运行容器）
  ~]# docker run [options] image[command]
  # 过滤出全部已经退出的容器并删掉
  ~]# for i in `docker ps -a|grep -i exit|awk '{print $1}'`;do docker rm -f $i;done
  # 查看日志，-f：跟踪日志输出，即是夯住，可以按ctrl+c
  ~]# docker log -f <容器id>
  
  Ctrl+c:强制中断程序的执行
  
  Ctrl+z:将任务中断,但是此任务并没有结束,他仍然在进程中他只是维持挂起的状态
  ```

* 容器常规操作

  ```shell
  映射端口
  docker run -p 容器外端口:容器内端口
  
  挂载数据卷
  docker run -v 容器外目录:容器内目录
  
  传递环境变量
  docker run -e 环境变量key:环境变量value
  
  查看内容
  docker inspect <容器id>
  
  容器内安装软件（工具）
  yum/apt-get/apt等
  
  --rm :用完即删
  --name：指定名字
  -d：放到后台，非交互式的
  ```

####  dockerfile

* dockerfile4组核心指令

  ```shell
  USER/WORKDIR指令
  ADD/EXPOSE指令
  RUN/ENV指令
  CMD/ENTRYPOINT指令
  ```

  

* dockerfile实验

  ```shell
  ~]# mkdir /data/dockerfile
  ~]# vi /data/dockerfile/Dockerfile
  FROM 909336740/nginx:v1.12.2
  USER root
  ENV WWW /usr/share/nginx/html
  ENV CONF /etc/nginx/conf.d
  RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&\ 
      echo 'Asia/Shanghai' >/etc/timezone
  WORKDIR $WWW
  ADD index.html $WWW/index.html
  ADD demo.od.com.conf $CONF/demo.od.com.conf
  EXPOSE 80
  CMD ["nginx","-g","daemon off;"]
  
  #构建镜像
  dockerfile]# docker build . -t 909336740/nginx:baidu
  
  FROM：从哪里导入
  USER：用什么用户起
  ENV：设置环境变量
  RUN： 修改时区成中国时区'Asia/Shanghai'
  WORKDIR：指定工作目录，这里指定的是之前ENV指定的WWW 目录，即是/usr/share/nginx/html
  ADD：添加指定的东西进去
  EXPOSE：暴露端口
  CMD：指令的首要目的在于为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束
  ```

* dockerfile四种网络类型

  ```shell
  Bridge contauner（NAT）   桥接式网络模式(默认)
  None(Close) container   封闭式网络模式，不为容器配置网络
  Host(open) container   开放式网络模式，和宿主机共享网络
  Container(join) container   联合挂载式网络模式，和其他容器共享网络
  ```


###  证书签发

####  证书类型

* ```shell
  client  certificate ：
  		客户端使用，用于服务端认证客户端，例如etcdctl、etcd proxy、 fleetctl、docker客户端
  server certificate：
  		服务端使用，客户端以此验证服务端身份，例如docker服务端。kube-apiserver
  peer certificate：
  		双向证书，用于etcd集群成员间通信
  ```

####  常用签发证书的工具

* 工具

  ```shell
  cfssl
  	主要用于签发证书
  cfssl-json
  cfssl-certinfo
  	查看证书有效信息
  openssl
  ```

* 命令

  ```shell
  # 签发证书
  [root@hdss7-200 certs]# cfssl gencert -initca ca-csr.json | cfssl-json -bare ca
  # gencert ：生成的意思
  [root@hdss7-200 certs]# cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem -config=ca-config.json -profile=peer etcd-peer-csr.json | cfssl-json  -bare etcd-peer
  
  # 查看证书相关信息
  [root@hdss7-200 certs]# cfssl-certinfo -cert apiserver.pem
  
  
  # 使用OpenSSL签发证书
  certs]# (umask 077; openssl genrsa -out dashboard.od.com.key 2048)
  # 没有openssl的yum install openssl
  certs]# openssl req -new -key dashboard.od.com.key -out dashboard.od.com.csr -subj "/CN=dashboard.od.com/C=CN/ST=BJ/L=Beijing/O=ben1234560/OU=ops"
  certs]# openssl x509 -req -in dashboard.od.com.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out dashboard.od.com.crt -days 3650
  certs]# cfssl-certinfo -cert dashboard.od.com.crt
  
  ```

####  需要证书的组件

* 需要证书的组件

  ```shell
  # etcd: 需要将etcd有可能部署到哪些组件的ip都填进来
  	类型 ：peer
  	需要的证书：ca.pem	etcd-peer.pem	etcd-peer-key.pem
  	
  # apiserver：需要将apiserver有可能部署到的ip都填进来
  	类型：server
  	需要的证书：ca.pem	ca-key.pem	client.pem	client-key.pem	apiserver.pem	apiserver-key.pem
  	
  # kubelet: 需要将kubelet有可能部署到的ip都填进来
  	类型：server
      需要的证书：ca.pem	ca-key.pem	client.pem	client-key.pem	kubelet.pem		kubelet-key.pem
  # kube-proxy:
  	类型：client
  	需要的证书：ca.pem	ca-key.pem	client.pem	client-key.pem	kube-proxy-client.pem
  kube-proxy-client-key.pem
  	
  #注意：部署kubelet和kube-proxy时的步骤是：set-cluster	-->	set-credentials	-->	set-context	-->	use-context；   会生成一个config文件，可以通过该config文件获取相关的证书。（获取阿里云证书）
  ]# echo "证书内容"  | base64 -d >> ca.pem
  	
  # flanneld: 实现宿主机里面容器的互通（为容器提供虚拟网络）
  	类型：client
  	需要的证书：ca.pem 	client.pem	client-key.pem	
  ```

####  检查组件成功的方法

* etcd

  ```shell
  方法一：
  [root@hdss7-22 etcd]# ./etcdctl cluster-health
  方法二：
  [root@hdss7-22 etcd]# ./etcdctl member list
  ```

  