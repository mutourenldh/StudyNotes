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

| 参数         | 释义        | 示例               |
| ---------- | --------- | ---------------- |
| -a         | 列出本地所有镜像层 | docker images -a |
| -q         | 只显示镜像ID   | docker images -q |
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

| 参数         | 释义                      | 示例   |
| ---------- | ----------------------- | ---- |
| --no-trunc | 显示完成的镜像描述               |      |
| -s         | 列出收藏不小于指定值的镜像           |      |
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

| 参数     | 释义                       |
| ------ | ------------------------ |
| --name | 为容器起一个别名                 |
| -d     | 后台运行容器，并且返回容器ID，即启动守护式容器 |
| -i     | 以交互模式运行容器，通常与-t同时使用      |
| -t     | 为容器重新分配一个输入伪终端，常与-i一起使用  |
| -P     | 随机端口映射                   |
| -p     | 指定端口映射，有以下四种格式           |

ip:hostPort:containerPort

ip::containerPort

hostPort:containerPort

containerPort

例子：

~~~shell
docker run -it 49f7960eb7e4	#启动交互式容器
docker run -d 容器ID或容器名 #启动守护式容器
~~~

![1532572018565](/images/1532572018565.png)

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

![1532575216595](/images/1532575216595.png)

![1532575233363](/images/1532575233363.png)

![1532576314306](/images/1532576314306.png)

#### 四、docker镜像

###### 1. 什么是镜像

镜像是一种轻量级的，可执行的独立软件包。用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码，运行时、库、环境变量和配置文件。

###### 2. 联合文件系统

![1532594400318](/images/1532594400318.png)

###### 3. docker镜像加载原理

![1532594471581](/images/1532594471581.png)![1532594481850](/images/1532594481850.png)

![1532594501669](/images/1532594501669.png)

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

![1532596258776](/images/1532596258776.png)

##### 2.作用

可以实现容器持久化

可以使容器间继承和共享数据

![1532596389508](/images/1532596389508.png)

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

![1532597040559](/images/1532597040559.png)

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

![1532598417347](/images/1532598417347.png)

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

![1532656312161](/images/1532656312161.png)

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

##### 2.基本知识

1.每个保留字指令都必须为大写字母并且后面要跟随至少一个参数

2.指令按照从上到下，顺序执行

3.#表示注释

4.每条指令都会创建一个新的镜像层，并对镜像进行提交。

##### 3.docker执行Dockerfile的大致流程

1.docker从基础镜像运行一个容器

2.执行一条指令并且对容器进行修改

3.执行类似docker commit的操作提交一个新的镜像层

4.docker再基于刚才提交的镜像运行一个新的容器

5.重复上述过程直到所有指令都执行完成

**总结**

![1532744851(1)](/images/1532744851(1).png)

![1532744912(1)](/images/1532744912(1).jpg**)

##### 4. Dockerfile体系结构（保留字）

**FROM：**+基础镜像，表示新镜像是基于哪个镜像来的

**MAINTAINER：**镜像维护者的姓名和邮箱地址

**RUN：**容器构建时需要运行的命令

**EXPOSE：**当前容器对外暴露的端口

**WORKDIR：**制定再容器创建之后，终端默认登陆进来的工作目录，一个落脚点

**ENV：**用来在构建镜像过程中设置环境变量

**ADD：**将宿主机目录下的文件拷贝至镜像并且add命令会自动 处理url和解压tar压缩包

**COPY：**类似ADD命令，拷贝文件和目录到镜像中。将从构建上下文目录中（源路径）的文件/目录复制到新的层镜像内的\<目标路径\>位置

格式：copy src dest

​	copy ["src","dest"]	

**VOLUME：**容器数据卷，用于数据保存和数据持久化的工作

**CMD：**制定一个容器启动的时候需要运行的命令，Dockerfile 中可以有多个CMD命令，但是只有最后一个生效，CMD会被docker run 之后参数所替换

注：CMD容器启动命令的格式和run相似，也是两种格式：

- shell格式：CMD命令
- exec格式：CMD ["可执行文件","参数1","参数2"...]
- 参数格式列表：CMD ["参数1","参数2"...]，在制定了ENTRYPOINT指令后，用CMD制定具体的参数

**ENTRYPOINT：**制定一个容器启动的时候要运行的具体命令，和CMD命令一样，都是制定容器启动时命令及参数

**ONBUILD：**当构建一个被继承的Dockerfile 时运行的命令，父镜像在被子继承后，父镜像的ONBUILD被触发

**总结：**

![2018-07-29_075100](/images/2018-07-29_075100.png)

##### 5. Dockerfile使用例子 

Docker Hub 中99%的镜像都是通过在base镜像中安装和配置需要的软件构建出来的

###### 1. 自定义镜像mycentos

**编写Dockerfile文件**

~~~dockerfile
FROM centos
MAINTAINER lidonghao<861914994@qq.com>
ENV MYPATH /usr/local
WORKDIR $MYPATH
RUN yum -y install vim
RUN yum -y install net-tools
EXPOSE 80
CMD echo $MYPATH
CMD echo "successful ........"
CMD /bin/bash
~~~

**构建**

~~~shell
docker build -f Dockerfile -t mycentos:1.3 .
docker build -f mycentos_dockerfile -t mycentos:1.3 .
~~~

构建成功

![1532827290(1)](/images/1532827290(1).png)

**运行**

~~~shell
docker  run -it mycentos:1.3
~~~

~~~shell
#列出镜像的变更历史命令
docker history 镜像名称
~~~

###### 2. CMD/ENTRYPOINT 

这两个命令都是制定一个容器启动 时需要运行的命令

**CMD命令**

Dockerfile中可以有多个CMD命令，但是只有最后一个会生效，CMD命令会被docker run 之后的参数替换

**测试：**tomcat的Dockerfile文件最后一行为`CMD ["catalina.sh", "run"]`,正常运行tomcat `docker run -it -p 8888:8080 tomcat`会打印tomcat的日志，但是我们运行命令	`docker run -it -p 8888:8080 tomcat ls -l` 则会列出当工作目录下的文件。这个时候 ls -l 即为docker run 之后的参数。

**ENTRYPOINT命令**

docker run 之后的参数会当作参数传递给ENTRYPOINT,之后形成新的命令组合。

**测试：**制作可以查询IP的容器

**编写Dockerfile文件**

~~~dockerfile
FROM centos
RUN yum install -y curl
CMD ["curl","-s","http://ip.cn"]
~~~

**构建**

~~~shell
docker build -f ip_dockerfile -t myip .
~~~

**运行**

~~~shell
docker run -it myip
~~~

**CURL命令说明：**

![2018-07-27_140929](/images/2018-07-27_140929.png)

如果我们还想显示文件头，这个时候就需要加上-i参数

~~~shell
curl -s -i http://ip.cn
~~~

但是这个时候在`docker run -it myip`之后加上 -i参数会报错，因为我们在dockerfile中使用的时CMD,加上 -i参数之后会替换 dockerfile 文件中的CMD命令。这个时候我们在dockerfile文件中改用ENTRYPOINT命令。他会将我们传的 -i参数拼接在 ENTRYPOINT 命令中形成新的命令，并且执行。

编写ENTRYPOINT版的dockerfile

~~~dockerfile
FROM centos
RUN yum install -y curl
ENTRYPOINT ["curl","-s","http://ip.cn"]
~~~

构建

~~~shell
docker build -f ip_dockerfile2 -t myip2 .
~~~

运行

~~~shell
docker run myip2		#不显示文件头
docker run myip2 -i   #显示文件头
~~~

###### 3. ONBUILD示例

构建父镜像：

（1）dockerfile

~~~dockerfile
FROM centos
RUN yum install -y curl
ENTRYPOINT ["curl","-s","http://ip.cn"]
ONBUILD RUN echo "this is father images........"
~~~

(2)构建

~~~shell
docker build -f father_dockerfile -t father .
~~~

(3)运行

~~~shell
docker run father
~~~

构建子镜像：

（1）dockerfile

```dockerfile
FROM father
RUN yum install -y curl
ENTRYPOINT ["curl","-s","http://ip.cn"]
```

(2)构建

```shell
docker build -f son_dockerfile -t son .
```

***构建子镜像的时候会执行父镜像dockerfile文件中的ONBUILD命令***

(3)运行

```shell
docker run son
```

###### 4. 自定义tomcat

1.编写Dockerfile文件(文件名称Dockerfile)

~~~dockerfile
#基础镜像为centos
FROM centos
#设置维护者及维护者邮箱
MAINTAINER lidonghao<861914994@qq.com>
#将aa.txt文件拷贝至容器中/usr/local文件夹下并且改名为mm.txt
COPY aa.txt /usr/local/mm.txt
#将tomcat和jdk压缩包拷贝至容器中/usr/local下并且解压
ADD apache-tomcat-8.0.26.tar.gz /usr/local/
ADD jdk-8u60-linux-x64.gz /usr/local/
#安装vim工具
RUN yum -y install vim
#设置环境变量
ENV MYPATH /usr/local
#制定工作目录
WORKDIR $MYPATH
#设置jdk和tomcat的环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_60
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-8.0.26
ENV CATALINA_BASE /usr/local/apache-tomcat-8.0.26
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#制定对外暴露端口为8080
EXPOSE 8080
#制定容器启动的时候启动tomcat并且打印日志
CMD /usr/local/apache-tomcat-8.0.26/bin/startup.sh && tail -F /usr/local/apache-tomcat-8.0.26/bin/logs/catalina.out
~~~

2.构建

~~~shell
docker build -t ldhtomcat9 .
~~~

这儿dockerfile文件名称为Dockerfile，所以不需要我们使用-f参数进行dockerfile文件的指定

3.运行

~~~shell
#-v设置容器卷
docker run -d -p 8089:8080 --name mytomcat99 -v /data/docker/app/mytomcat9/test:/usr/local/apache-tomcat-8.0.26/webapps/test -v /data/docker/app/mytomcat9/logs/:/usr/local/apache-tomcat-8.0.26/logs --privileged=true ldhtomcat9
~~~

4.查看

~~~shell
docker exec e117d8c99d9c java -version
docker exec e117d8c99d9c pwd
docker exec e117d8c99d9c ls -l
docker exec e117d8c99d9c cat mm.txt
~~~

5.web项目的发布

在/data/docker/app/mytomcat9/test目录下分别创建a.jsp文件和WEB-INF文件夹，在WEB-INF文件夹中创建web.xml文件

web.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
<display-name>test</display-name>
</web-app>
~~~

a.jsp

~~~jsp
<%@ page language="java" contentType="text/html; charset=utf-8" pageEncoding="utf-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>

<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>测试页面</title>
</head>
<body>
----------------welocme--------------------
<%="I am in docker tomcat"%>
  <br>
  <br>
  <% System.out.println("Hello World");%>
  </body>
</html>
~~~

然后重启mytomcat99`docker restart d0a77d8ce634`

浏览器访问http://47.105.103.45:8089/test/a.jsp

###### 5.docker上安装mysql

~~~shell
docker run -p 3306:3306 --name mysql \
-v /data/docker/app/mysql5.6/conf:/etc/mysql/conf.d \
-v /data/docker/app/mysql5.6/logs:/logs \
-v /data/docker/app/mysql5.6/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
~~~

![2018-07-27_151904](/images/2018-07-27_151904.png)

安装完执行`docker exec -it 15a4658343ff /bin/bash`进入交互式

~~~shell
mysql -uroot -p   #连接mysql
~~~

数据库备份命令测试

~~~shell
docker exec 15a4658343ff sh -c 'exec mysqldump --all-databases -uroot -p "123456"'> /data/docker/app/mysql5.6/all-database.sql
~~~

6.docker上安装redis

~~~shell
docker run -p 6379:6379 -v /data/docker/app/myredis/data:/data -v /data/docker/app/myredis/conf/redis.conf:/usr/local/etc/redis/redis.conf -d redis redis-server /usr/local/etc/redis/redis.conf --appendonly yes
~~~

在主机/data/docker/app/myredis/conf/redis.conf文件夹下新建文件redis.conf。这个是redis的配置文件

连接redis:docker exec -it 7c4d512635a9 redis-cli

退出redis   shutdown

/data/docker/app/myredis/data 目录下是redis中数据存放的位置。

#### 七、将本地镜像推送到阿里云

##### 1.根据容器创建镜像

~~~shell
docker commit -a ldh -m "new vim centos 1.4" b5ba2cd1f8c7 mycentos:1.4
~~~

查看我们新生成的镜像

dockers ps

运行 docker run -it mycentos:1.4

##### 2.将本地镜像推送至阿里云	

登陆阿里云镜像仓库管理控制台https://cr.console.aliyun.com

创建镜像仓库的时候代码源暂时选择本地仓库

步骤：
1.登陆阿里云

~~~shell
sudo docker login --username=lidonghao4 registry.cn-hangzhou.aliyuncs.com
~~~

2.

~~~shell
sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/lidonghao/mycentos:[镜像版本号]   #命令模板
~~~

~~~shell
docker tag 532033853ef3 registry.cn-hangzhou.aliyuncs.com/lidonghao/mycentos:1.4		#命令示例
~~~

3.

~~~shell
sudo docker push registry.cn-hangzhou.aliyuncs.com/lidonghao/mycentos:[镜像版本号]	#命令模板
~~~

~~~shell
#命令示例
docker push registry.cn-hangzhou.aliyuncs.com/lidonghao/mycentos:1.4
~~~

