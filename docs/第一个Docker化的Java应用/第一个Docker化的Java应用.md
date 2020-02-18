【第一个Docker化的Java应用】
快速的持续集成，服务的弹性伸缩等，部署简单，解放运维，节省机器资源。2015年618月京东有15万个Docker实例，并且所有实例全部Docker化。

【Docker简单认识】
Docker id the world's leading software containerization platform.托管在github上。跨平台，支持Windows、Macos、Linux。
Docker是一个用来装应用的容器，就像杯子可以装水，笔筒可以放笔，书包可以放书一样，可以把任何想得到的程序放到docker里。
Docker官网：www.docker.com

Docker很多命令都跟Git相似。


【理解Docker】
Docker思想：
	·集装箱
	·标准化：运输方式、存储方式（不需关心应用存在什么地方）、API接口（提供了一系列的REST API接口）。
	·隔离：类似于虚拟机，但是更轻量。可实现快速创建和销毁，最底层的技术实际上是Linux的一种内核限制机制LXC。
			LXC是一种轻量级的容器虚拟化技术，最大效率的隔离了进程和资源，通过cgroup、namespace的限制，隔离进程组所使用的物理资源，比如cpu\memory\io等等。
			
Docker出现解决了什么问题？
	一个Java Web正常运行需要依赖的有：最底层的操作系统、操作系统上依赖JDK、tomcat、代码、配置文件。
	docker把所有依赖的东西都放到集装箱里，再打包放到鲸鱼上，由鲸鱼给送到服务器上，在服务器上运行。
	
	共用服务器问题，会出现程序挂掉，不是内存满了就是硬盘满了。或者发现某个服务变慢了，甚至敲终端都非常卡，可Linux本身就是多租户操作系统，可以供多用户同时使用，怎么办呢？--Docker的隔离性完全解决了这个问题。
	把大家程序都放在Docker里运行，就算别人的程序还是死循环，疯狂吃CPU或者打日志占满磁盘或者占用大量内存内存泄漏，都不会导致别人的程序运行错误，只会导致自己的程序。
	因为Docker在启动的时候就给它限定好了最大使用的CPU、内存、硬盘，如果超过了你就会被杀掉。
	
	业务量并不是平均的，需要按需伸缩节点。使用Docker能瞬间扩增几十甚至几百台服务器出来。
	
	1.Docker解决了运行环境不一致所带来的问题。
	2.解决了服务器共用带来的资源使用问题。
	3.Docker的标准化让快速扩展、弹性伸缩变得简单。

	
【走进Docker】
#####Docker核心技术：镜像、仓库、容器
核心就是：构建镜像、运行容器，镜像通过仓库来传输。
Docker集装箱的思想：体现在镜像上，集装箱是分层的，每一层的东西都是固定的，一个镜像打好后就好像一个大大的集装箱，它里面的东西是固定的文件，不会发生变化。
Docker标准化思想：启动、下载、运行镜像的方式都会发现所做的操作都是基本差不多的。
Docker隔离思想：网络的隔离提供了一个进程的隔离、提供了一个磁盘的隔离。让容器运行起来会拥有自己的进程空间，并且他还有自己的磁盘、网络，因为容器内的网络外面是访问不到的。

1.Build（镜像--集装箱）				----构建镜像（目的是在其他机器、环境运行程序）
2.Ship（仓库--超级码头）			----运输镜像
3.Run（容器--运行程序的地方）		----运行的镜像
用Docker运行一个程序的过程：去仓库把镜像拉倒本地，然后用一条命令把镜像运行起来变成容器。

#####Docker镜像
镜像（image），鲸鱼驮着的所有集装箱就是一个镜像。本质上讲，其实镜像就是一系列的文件，可以包括应用程序的文件，也可以包括应用的运行环境的文件。
既然都是文件，Docker肯定把它保存在了本地。那是以什么格式保存的呢？涉及到Linux的存储技术叫做联合文件系统UnionFS，他是一种分层的文件系统，可以将不同的目录挂到同一个虚拟文件系统下。
Docker镜像正是利用了Linux的分层概念来实现的镜像存储。
Docker镜像每一层文件系统都是只读的，把每一层加载完成之后，这些文件都会被看做是同一个目录，相当于只有一个文件系统。Docker的这种文件系统就被称作镜像。

#####Docker容器
容器的本质就是一个进程。为了便于理解，可以先把容器想象成一个虚拟机。
每个虚拟机都有自己的文件系统，所以可以把Docker镜像看做是Docker容器的文件系统。
Docker容器的文件系统与虚拟机的文件系统的区别在于：
	Docker容器的文件系统是一层层的，并且最下面的N层都是只读的，只有最上面一层是可写的。
		·当大家的程序运行起来，势必要写一些日志，写一些文件，或者是对系统的某一些文件做一些修改，所以容器在最上面一层创建了一个可读可写的文件系统。	
		·程序在运行过程中如果去写一个镜像中的文件，因镜像的每一层都是只读的，所以它会在写这个文件之前把这个文件这一层考到容器的最上层，然后再对它进行修改。
	修改后，当我们的应用读一个文件的时候，它会从最顶层开始查找，如果没有才会找下一层。
	
	由于容器的这一层是可以修改的，而镜像是不可以修改的，这样可以保证：同一个镜像可以生成多个容器独立运行，而它们之间没有任何的干扰。

#####Docker仓库
构建镜像的目的是把他传递到其他机器、环境去运行。
传输过程则使用到了Docker仓库，要先把Docker镜像传到Docker仓库中，再由目的地去Docker仓库把Docker镜像拉过去，这就完成了传输过程。
Docker仓库肯定会提供一个中央服务器，给一个地址去访问它。
Docker自己hub.docker.com提供了Docker仓库服务。但其下载镜像很慢，国内也在做自己的仓库，比较知名的有c.163.com（网易蜂巢的镜像中心）.
如果镜像比较私密，Docker也支持自己搭建镜像中心。

【Docker Windows安装】
1.安装DockerToolbox
双击安装包-Next-选择一个安装位置，点击Next-默认安装就可以了-点击始终信任oracle的安装。
2.安装后，点击Docker Quickstart Terminal，第一次运行时间会长一点。
它会去找boot2docker.iso。我们需要把下载好的boot2docker.iso拷贝到用户的根目录(如)C:\Users\jianghongyu.GIS\.docker\machine\cache下，也即命令行提示我们的目录。
再次点击Docker Quickstart Terminal。
3.检测安装。
在安装过docker的终端输入docker version。如果能显示Client、Server的信息则表示安装成功。


【Docker Linux安装】
Docker本身在Ubuntu系统上开发的，对其支持最好
Linux系统的发行版本比较多。
	·Redhat&CentOS（同源）：系统要求：64-bit OS and version 3.10（内核版本）参考www.imooc.com/article/16448
	·Ubuntu：系统要求：64-bit OS and version 3.10（内核版本）
Linux内核需在3.10以上。


【Docker实践部分】
#####第一个docker镜像helloworld
[下载]
·docker pull [OPTIONS] NAME[:TAG]				从docker远程仓库拉取一个镜像到本地，NAME表示要拉取的镜像的名称，TAG可选，如果不加会默认加上:latest表示最新版本。OPTIONS是拉取的一些参数。
·docker images [OPTIONS] [REPOSITORY[:TAG]]		查看本机有哪些镜像，也可验证pull是否成功了。指定镜像的名称和版本。
[运行]
·docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...]	COMMAND表示镜像在运行起来的时候要执行什么命令，ARG表示命令所需的参数
[使用示例]
	docker images查看本机所有镜像列表，发现没有任何镜像，只打印了每一列的列头.其中的IMAGE ID是一个64wei字符串可唯一标识镜像
	docker pull hello-world没有指定网站去拉取镜像，会默认到Docker提供的docker仓库去把镜像下载回来。
	docker run hello-world（前台启动一个镜像）


【Docker运行Nginx Web服务器】
#####Nginx镜像的特点
·持久运行的容器
·前台挂起&后台运行   前台运行的镜像可以用Ctrl+C结束进程，Nginx最好使用后台运行
·进入容器内部

去网易蜂巢镜像中心查nginx的镜像名称，复制下载地址
docker pull hub.c.163.com/library/nginx:latest拉取镜像
docker images	查看所有镜像
docker ps 查看目前正在这台机器上运行的容器进程
docker run --help查看参数
docker run -d hub.c.163.com/library/nginx后台运行容器（去掉-d参数即在前台运行容器）
docker exec --help在一个运行的容器中运行一个命令
docker exec -it f4 bash 分配一个伪终端，我们才回去输入我们的内容(指定容器的id或者名字)
	进去后可以使用ls,which nginx,ps -ef查看当前服务都有哪些进程,exit

【Docker网络--->在主机浏览器中访问到运行的Nginx】
#####网络类型
	Bridge桥接
	Host
	None
Docker具有隔离性，网络也是隔离性的一部分。
Linux用了namespace即命名空间来进行资源的隔离，比如PID namespace就是用来隔离进程，mark namespace就是用来隔离文件系统的，network namespace就是用来隔离网络的。
每一个network namespace都提供了一份独立的网络环境，包括网卡、路由、iptables规则等等。
·Docker容器一般默认情况下回分配一个独立的namespace，也就是网络类型中的bridge模式。
·还有一种网络类型是Host模式，如果启动容器时候指定使用Host模式，那么容器将不会获得一个独立的network namespace，而是和主机共同使用一个。这时容器将不会再虚拟出自己的网卡、配置出自己的IP等。而是会使用宿主机上的IP和端口。
·还有一种网络类型是没有网络，Docker将不会跟外界的任何东西进行通讯。
	
	
#####端口映射
当使用Bridge模式时候，就涉及到一个问题：既然它使用的网络是独立的namespace，这就需要一种技术（端口映射）使容器中的端口能够在宿主机上访问到。
Docker可以指定你想把容器内的某一个端口和容器所在主机上的某一个端口之间进行一个映射，当你在访问主机上这个端口的时候，其实就是访问容器里面的那一个端口。

docker默认使用bridge模式，原来在该模式时并没有告诉它端口如何映射，所以主机上访问不到容器里的端口。

docker stop	容器的id或名字							停止指定镜像
docker run -d -p 8080:80 容器id或名字				主机端口:容器端口
netstat -na|grep 8080								检测是否开放了端口
docker run -d -P hub.c.163.com/library/nginx		开放所有监听端口映射到主机的随机端口


【制作自己的镜像】
去网易163镜像中心找一个tomcat镜像作为我们的基础镜像。
docker pull hub.c.163.com/library/tomcat:latest		tomcat这个镜像肯定包括JDK的。
[写Dockerfile]
vi Dockerfile
from hub.c.163.com/library/tomcat				from 基础镜像的名字--告诉docker做一个自己的镜像，是以tomcat镜像为起点。
MAINTAINER jianghy xxx@126.com					镜像所有者的名字
COPY jpress.war /usr/local/tomcat/webapps		把web应用放到镜像里面，此处是tomcat的webapps下(具体路径查看镜像下载页面中的CATALINA_HOME) COPY 本地文件
docker build -t jpress:latest .					构建镜像，参数跟Dockerfile的存放目录，当前目录是一个点

修改文件的名字 mv old文件 new文件

【运行自己的容器】
docker run -d -p 8888:8080 jpress
docker ps
netstat -na|grep 8888
浏览器访问localhost:8888/jpress

docker的-e命令是指定环境变量
在镜像中心搜索mysql的地址，下载下来MySQL镜像,启动MySQL镜像并创建MySQL数据库
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=000000 -e MYSQL_DATABASE=jpress hub.c.163.com/library/mysql:latest 
docker ps
netstat -na|grep 3306
启动后浏览器访问localhost:8888/jpress
在JPress安装向导里进行配置
配置、安装完成后重启web服务器  docker restart 容器ID

【总结】
下拉镜像 
构建镜像、运行容器，镜像通过仓库来传输。运行容器
停止容器
重启容器
进入容器内部



