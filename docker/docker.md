#### 一、docker简介

docker是一种解决了运行环境和配置问题的软件容器，方便做持续集成并有助于整体发布的容器虚拟化技术。

![1532412689425](/images/425.png)

![1532412698147](/images/147.png)

![1532412708581](/images/581.png)

###### 1. 虚拟机技术和容器虚拟化技术比较：

![1532412747263](/images/263.png)

![1532412779114](/images/114.png)

![1532412808826](/images/826.png)

###### 2. 容器虚拟化技术优点：

- 更快速的应用交付和部署
- 更便捷的升级和扩容
- 更简单的运维系统
- 更高效的计算资源利用

#### 二、docker安装

docker官网：www.docker.com

docker中文网：https://www.docker-cn.com/

##### 1. 环境支持

docker支持的Centos版本

Centos7(64-bit)，系统内核版本为3.10以上

Centos6.5(64-bit)或更高的版本  ，系统内核版本为2.6.32-431或者更高的版本

###### 1. 查看系统版本信息

两种方式

~~~shell
lsb_release -a
~~~

![1532413825875](/images/875.png)



~~~she
cat /etc/redhat-release
~~~

![1532414040243](/images/243.png)

###### 2. 查看系统内核版本

~~~shel
uname -r
~~~

![1532413790808](/images/808.png)

##### 2. 重要概念

docker架构图

![1532414393052](/images/052.png)

###### 1.镜像

![1532414464651](/images/651.png)

###### 2.容器

![1532414541045](/images/045.png)

###### 3.仓库

![1532414670848](/images/848.png)

总结：

![1532414690877](/images/877.png)

##### 3. docker的安装

###### 1. centos6.8安装docker

![1532417720903](/images/903.png)

~~~shell
yum install -y epel-release
yum install -y docker-io
service docker start		 #重启docker服务
docker version				#检查docker版本
~~~

注：docker使用EPEL发布，RHEL系的OS首先要保证已经有EPEL仓库，否则先检查OS的版本，然后安装相应的EPEL包

###### 2. centos6.8配置阿里云镜像加速

加速地址获取

https://cr.console.aliyun.com/?spm=5176.1971733.0.2.6c045aaa6jqmCd#/accelerator

自己的阿里云镜像加速

专属加速地址：https://fvrybxjp.mirror.aliyuncs.com

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://fvrybxjp.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

修改配置文件 /etc/sysconfig/docker

![1532418631745](/images/745.png)

重启docker服务

~~~shell
service docker restart
~~~

###### 3. centos 7安装

官网：https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce

中文版：https://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#%E5%8D%B8%E8%BD%BD-docker-ce（不一定是最新的）

3.1 安装gcc相关

~~~shell
yum -y install gcc
yum -y install gcc-c++
gcc -v    #查看gcc版本
~~~

3.2 卸载旧版本

```shell
 #如果是系统root权限，则不用写sudo
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

3.3 安装所需的软件包。`yum-utils` 提供了 `yum-config-manager` 实用程序，并且 `devicemapper`存储驱动需要 `device-mapper-persistent-data` 和 `lvm2`。 

~~~shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
~~~

3.4 使用下列命令设置 **stable** 镜像仓库。您始终需要使用 **stable** 镜像仓库 

~~~shell
#国外地址    \相当于换行的连接符
sudo yum-config-manager \
     --add-repo \
     https://download.docker.com/linux/centos/docker-ce.repo
~~~

~~~shell
sudo yum-config-manager --add-repo \
https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
~~~

3.5 更新 `yum` 软件包索引。 

~~~shell
sudo yum makecache fast
~~~

3.6 安装DOCKER CE

~~~SHELL
 sudo -y yum install docker-ce
~~~

3.7 启动docker

~~~shell
sudo systemctl start docker
~~~

3.8 测试

~~~shell
sudo docker run hello-world   #运行hello world
docker version   #查看docker版本
~~~

3.9 配置镜像加速

~~~shell
mkdir -p /etc/docker
vim /etc/docker/daemon.json
~~~

在daemon.json中添加

~~~shell
{
  "registry-mirrors": ["https://fvrybxjp.mirror.aliyuncs.com"]
}
~~~

然后执行以下命令

~~~shell
#重新加载配置文件
systemctl daemon-reload
#重启docker
systemctl restart docker
~~~

3.10 卸载docker

~~~shell
#停止docker
sudo systemctl stop docker
#卸载 Docker 软件包
sudo yum remove docker-ce
#主机上的镜像、容器、存储卷、或定制配置文件不会自动删除。如需删除所有镜像、容器和存储卷，请运行下列命令
sudo rm -rf /var/lib/docker
~~~

##### 4. docker run

~~~shell
docker run hello-world
~~~

docker run 命令工作过程

![1532504138505](/images/505.png)

![1532504707267](/images/267.png)

![1532504719636](/images/636.png)

![1532504731232](/images/232.png)

![1532504744351](/images/351.png)



#### 三、docker常用命令

##### 1. 帮助命令

~~~shell
docker version
docker info
docker --help
~~~

##### 2. 镜像命令

###### 1. docker images

~~~shell
#显示本地镜像
docker images
~~~

参数：

| 参数       | 释义               | 示例             |
| ---------- | ------------------ | ---------------- |
| -a         | 列出本地所有镜像层 | docker images -a |
| -q         | 只显示镜像ID       | docker images -q |
| --digests  | 显示镜像的摘要信息 |                  |
| --no-trunc | 显示完整的镜像信息 |                  |

组合命令

docker images -qa    返回本地所有镜像的ID

![1532504887970](/images/970.png)

###### 2. docker search

~~~shell
#从docker hub查询指定镜像信息
docker search [OPTIONS]镜像名称
~~~

| 参数       | 释义                            | 示例 |
| ---------- | ------------------------------- | ---- |
| --no-trunc | 显示完成的镜像描述              |      |
| -s         | 列出收藏不小于指定值的镜像      |      |
| -automated | 只列出automated build类型的镜像 |      |

~~~shell
docker search -s 30 tomcat
~~~

###### 3.docker pull

~~~shell
#从仓库中拉取镜像（默认拉取docker hub上的，配了阿里云加速之后从阿里云仓库下载）
docker pull 镜像名称[:TAG]
#示例
docker pull tomcat  
#默认下载最新的，等价于 docker pull tomcat:latest
~~~

###### 4. docker rmi

~~~shell
docker rmi -f 镜像ID	#删除单个镜像
docker rmi -f 镜像名1:TAG 镜像名2:TAG	#删除指定多个镜像
docker rmi -f $(docker images -qa)	#删除全部镜像
~~~

-f  表示强制删除

示例：

~~~shell
docker rmi -f hello-world
docker rmi -f hello-world nginx
~~~

##### 3. 容器命令

###### 1.docker run  

###### 新建并启动一个容器

参数：

| 参数   | 释义                                           |
| ------ | ---------------------------------------------- |
| --name | 为容器起一个别名                               |
| -d     | 后台运行容器，并且返回容器ID，即启动守护式容器 |
| -i     | 以交互模式运行容器，通常与-t同时使用           |
| -t     | 为容器重新分配一个输入伪终端，常与-i一起使用   |
| -P     | 随机端口映射                                   |
| -p     | 指定端口映射，有以下四种格式                   |

ip:hostPort:containerPort

ip::containerPort

hostPort:containerPort

containerPort

例子：

~~~shell
docker run -it 49f7960eb7e4	#启动交互式容器
docker run -d 容器ID或容器名 #启动守护式容器
~~~

![1532572018565](E:\学习笔记\docker\1532572018565.png)

###### 2. docker ps

列出当前正在运行中的容器

- -a :列出当前正在运行的容器和历史上运行过的所有容器

- -l：显示最近创建的容器

- -n：显示最近n个创建的容器

- -q：静默模式，只显示容器编号

- --no-trunc：不截断输出

  ~~~shell
  docker ps -ql
  docker ps -n 2
  ~~~

###### 3. 退出容器

两种方式

- exit ：容器退出并且停止
- ctrl+P+Q：容器不停止退出

###### 4. 启动容器

docker start+容器ID或者名称

###### 5. 停止容器

docker stop 容器ID或容器名称

###### 6.强制停止容器

docker kill 容器ID或者容器名称

###### 7. 删除容器

docker rm -f 容器ID或者容器名称

删除多个容器

~~~shell
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
~~~

###### 8. 查看容器日志

docker logs -f -t --tail 容器ID

-f ：跟随最新的日志打印

-t：加入时间戳

--tail 数字：显示最后多少条

~~~shell
docker logs -f -t --tail 5 f886265e1980
~~~

###### 9. 查看容器内运行的进程

docker top 容器ID

###### 10. 查看容器内部细节

docker inspect 容器ID

###### 11. 进入正在运行中的容器以命令行进行交互

~~~shell
#在容器中打开新的终端，并且可以启动新的进程
docker exex -it 容器ID baseShell
#重新进入容器启动命令的终端，不会启动新的进程
docker attach 容器ID	
~~~

示例：

~~~shell
docker run -it centos /bin/bash   #以交互式启动centos
ctrl+P+Q						#不停止退出
#不进入centos,在宿主机上执行命令操作centos
docker exec -t f48a49415cd8 ls -l	
docker attach f48a49415cd8	#重新进入启动中的centos
~~~

###### 12. 从容器内拷贝文件到主机上

docker cp 容器ID：容器内路径 目的路径

~~~shell
docker cp f48a49415cd8:/tmp/yum.log /root
~~~

###### 13 常用命令总结

![1532575216595](E:\学习笔记\docker\1532575216595.png)

![1532575233363](E:\学习笔记\docker\1532575233363.png)

![1532576314306](E:\学习笔记\docker\1532576314306.png)

#### 四、docker镜像

###### 1. 什么是镜像

镜像是一种轻量级的，可执行的独立软件包。用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码，运行时、库、环境变量和配置文件。

###### 2. 联合文件系统

![1532594400318](E:\学习笔记\docker\1532594400318.png)

###### 3. docker镜像加载原理

![1532594471581](E:\学习笔记\docker\1532594471581.png)![1532594481850](E:\学习笔记\docker\1532594481850.png)

![1532594501669](E:\学习笔记\docker\1532594501669.png)

###### 4. docker镜像采用分层结构好处

可以实现资源共享，比如多个镜像从相同的base镜像而来，则宿主机上只需要保存一份base镜像即可。同时内存中也只需要加载一份base镜像，就可以为所有的镜像服务了。而且镜像的每一层都可以被共享。

docker镜像的特点：

docker镜像都是只读的，当容器启动的时候，一个新的可写层被加载到镜像的顶部。这一层通常称作"容器层"，“容器层”之下的都叫"镜像层".

###### 5. docker镜像commit操作

docker commit 可以提交容器副本使之成为一个新的镜像

命令：

~~~shell
docker commit -m="" -a="" 容器ID 要创建的目标镜像名称：[标签名]
~~~

示例：将tomcat中的doc文档删除之后以他为模板commit一个新镜像

~~~shell
#以交互式运行tomcat
docker run -it -p 8888:8080 tomcat
#退出不停止该tomcat
ctrl+P+Q
#重新进入该tomcat
docker exec -it 9ee76ad893c8 /bin/bash
cd webapps		#进入webapps
ls -l			#列出当前目录

rm -rf docs		#删除docs
ctrl+P+Q		#退出不停止该tomcat
#以这个删除docs的tomcat（id为9ee76ad893c8）为模板创建镜像，lidonghao 为命名空间，1.0为版本号
docker commit -a='ldh' -m='del docs tomcat' 9ee76ad893c8 lidonghao/tomcat01:1.0
#运行我们生成的镜像，该tomcat没有docs
docker run -d -p 8887:8080 lidonghao/tomcat01:1.0
#运行原来的tomcat，该tomcat有docs
docker run -d -p 8886:8080 tomcat
~~~

#### 五、docker容器数据卷

##### 1.docker容器数据卷介绍

有点类似我们redis中的rdb和aof文件

![1532596258776](E:\学习笔记\docker\1532596258776.png)

##### 2.作用

可以实现容器持久化

可以使容器间继承和共享数据

![1532596389508](E:\学习笔记\docker\1532596389508.png)

##### 3.容器内添加数据卷

###### 1.命令行添加

**命令行格式**

~~~shell
docker run -it -v /宿主机绝对路径目录：/容器内目录 镜像名
~~~

~~~shell
#以交互式启动centos，并且添加数据卷
docker run -it -v /hostVolume:/containerVolume centos
~~~

**查看数据卷是否挂载成功**

~~~shell
docker inspect 6cd97d1e3261
~~~

![1532597040559](E:\学习笔记\docker\1532597040559.png)

最终效果：宿主机上的hostVolume和容器中的containerVolume中的文件完全同步。

我们可以在hostVolume和containerVolume中分别创建不同的文件，最终发现两个文件夹中的文件完全同步。并且即使容器停止，我们修改宿主机hostVolume文件夹中的文件，等容器再次启动的时候hostVolume中的改动依然可以同步到containerVolume中。

**测试流程：**

1.退出启动中的centos :exit

2.修改hostVolume文件夹中的文件

3.执行 `docker ps -a ` `docker start 容器ID`  `docker attach 容器ID`

4.查看containerVolume文件夹下的文件，发现和hostVolume中的同步

**加带权限的数据卷**

~~~shell
docker run -it -v /hostVolume:/containerVolume：ro centos
~~~

*ro： read only*

这个时候我们容器中的文件夹containerVolume中的文件只能依据hostVolume中的文件进行同步，而我们不可以手动修改containerVolume中的文件或者在containerVolume中创建新的文件。

**查看带权限的数据卷是否挂载成功**

![1532598417347](E:\学习笔记\docker\1532598417347.png)

**常见错误**

如果docker挂载主机目录Docker访问出现cannot open directory: Permission denied

解决办法：在挂载目录后加一个	--privileged=true参数即可

~~~shell
docker run -it -v /hostVolume:/containerVolume --privileged=true centos 
~~~

###### 2.DockerFile添加

**操作步骤**

1.在根目录下新建mydocker文件夹并进入

2.在mydocker中新建dockerFile文件，并且编辑

~~~shell
#volume test
FROM centos
VOLUME ["/dataContainerVolume1","/dataContainerVolume2"]
CMD echo "finished,...................successful"
CMD /bin/bash
~~~

说明：

处于可移植和分享的考虑。用-v 主机目录：容器目录这种方式不可以直接在dockerFile中实现。

由于宿主目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都有这样特定的目录。

3.build创建新的镜像

~~~shell
#在当前目录下使用dockerFile文件创建lidonghao/centos01镜像
docker build -f /mydocker/dockerFile -t lidonghao/centos01 .
~~~

![1532656312161](E:\学习笔记\docker\1532656312161.png)

##### 4.数据卷容器

###### 1.数据卷容器是什么

数据卷容器是命名的容器挂载数据卷，其他容器通过挂载这个（父容器）实现数据共享，挂载数据卷的容器，称之为数据卷容器。

###### 2.作用

可以实现容器间的传递共享

###### 3.测试

**启动一个父容器**

~~~shell
docker run -it --name dc01 lidonghao/centos01
~~~

**分别启动dco2,dc03,dc04继承自前一容器**

docker run -it --name dc02 --volumes-from dc01 lidonghao/centos01

docker run -it --name dc03 --volumes-from dc02 lidonghao/centos01

docker run -it --name dc04 --volumes-from dc03 lidonghao/centos01

他们都有数据卷dataContainerVolume1，dataContainerVolume2

我们在dc01中的dataContainerVolume1创建aa.txt。发现dc02,dc03,dc04中都有aa.txt。执行`docker rm -f dc02`  修改dc03中的aa.txt，发现dc01,dc04中的aa.txt也得到了同步。

**结论：**容器之间配置信息的传递，数据卷的生命周期一直持续到没有容器使用它为止。

#### 六、Dockerfile解析

##### 1.概述

Dockerfile是用来构建Docker镜像的构建文件，是一系列命令和参数构成的脚本。

**构建步骤：**
编写Dockerfile文件----docker build-----docker run 

**centos的Dockerfile文件:**

~~~dockerfile
FROM scratch
MAINTAINER The CentOS Project <cloud-ops@centos.org>
ADD c68-docker.tar.xz /
LABEL name="CentOS Base Image" \
    vendor="CentOS" \
    license="GPLv2" \
    build-date="2016-06-02"

# Default command
CMD ["/bin/bash"]
~~~

