![1565192348193](D:\笔记\note\docker\1565192348193.png)

## 1.centos7 安装docker

1. yum -y install gcc

2. yum -y install gcc-c++

3. 卸载旧版本

   yum -y remove docker docker-common docker-selinux docker-engine

4. 安装需要的软件包

   yum install -y yum-utils device-mapper-persistent-data lvm2

5. 设置stable镜像仓库

   yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

6. 更新yum软件包索引

   yum makecache fast

7. 安装docker

   yum -y install docker-ce

8. 启动

   systemctl start docker

9. 配置镜像加速

   mkdir -p /etc/docker

   vim  /etc/docker/daemon.json

   ```
   {
     "registry-mirrors": ["https://q8prvcuq.mirror.aliyuncs.com"]
   }
   ```

   systemctl daemon-reload

   systemctl restart docker

10. 卸载

    systemctl stop docker 

    yum -y remove docker-ce

    rm -rf /var/lib/docker



## 2.镜像命令

**1. docker images [command]**

```
-a :列出本地所有的镜像（含中间映像层）
-q :只显示镜像ID
--digests :显示镜像的摘要信息
--no-trunc :显示完整的镜像信息
```

**2. docker search [command] 镜像名**

```
--no-trunc : 显示完整的镜像描述
-s : 列出收藏数不小于指定值的镜像
--automated : 只列出 automated build类型的镜像
```

**3. docker pull 镜像名:[tag]**

tag 默认是latest

**4. docker rmi 镜像名**



## 3. 容器命令

**docker run [OPTIONS] IMAGE [COMMAND] [ARG...]**

```
--name="容器新名字": 为容器指定一个名称；
-d: 后台运行容器，并返回容器ID，也即启动守护式容器；
-i：以交互模式运行容器，通常与 -t 同时使用；
-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；
-P: 随机端口映射；
-p: 指定端口映射，有以下四种格式
	  hostPort:containerPort
      ip:hostPort:containerPort
      ip::containerPort
      containerPort
```

前台启动，name 可不指定，这样就直接启动了 Tomcat

docker run --name mytomcat -it tomcat

后台启动

docker run --name mytomcat -d tomcat

指定端口启动，宿主机的 8080 端口对应 Tomcat 的 8080

docker run --name mytomcat -d -p 8080:8080 tomcat

若要直接进入容器，而不启动tomcat

docker run -it tomcat /bin/bash

**docker ps [options]**

```
-a :列出当前所有正在运行的容器+历史上运行过的
-l :显示最近创建的容器。
-n：显示最近n个创建的容器。
-q :静默模式，只显示容器编号。
--no-trunc :不截断输出。
```

**exit** 退出容器并结束容器

**ctrl+P+Q** 退出但是不结束容器

docker start 容器id	启动一个已经关闭了的容器

docker restart 容器id 重启容器

docker stop 容器id 停止容器

docker kill 容器id 强制定制容器

docker rm 容器id，删除已停止的容器

* 删除多个 docker ps -aq | xargs docker rm

docker logs -f -t --tail 20 容器ID

```
   -t 是加入时间戳
   -f 跟随最新的日志打印
   --tail 数字 显示最后多少条
```

docker top 容器ID，查看容器内的进程

docker inspect 容器ID ，查看容器的细节



docker attach 容器id，进入正在运行的容器中

docker exec -it 容器ID /bin/bash，进入正在运行的容器，但是会新开一个进程

docker exec 容器id tail -f /usr/local/tomcat/logs/，执行一个命令，把结果返回到宿主机

docker cp 容器id : 容器路径  宿主机目录，复制容器内文件到宿主机



## 4. docker 镜像

docker commit -a=”loxn“ -m “del tomcat docs”  镜像id 镜像名称：版本	提交镜像到本地镜像仓库

```xml
docker commit -a='loxn' -m='del docs' 4fc97a1ebce1 loxn/tomcat:1.1
```



## 5. docker 容器数据卷

​	卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：

​	卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：

1：数据卷可在容器之间共享或重用数据

2：卷中的更改可以直接生效

3：数据卷中的更改不会包含在镜像的更新中

4：数据卷的生命周期一直持续到没有容器使用它为止



**1. 命令添加卷，容器重启后，数据依然同步**

docker run -it -v /宿主机路径：/容器内目录 镜像id

docker run -it -v /宿主机路径：/容器内目录:**ro** 镜像id，readonly，容器只能读不能写

**2.使用dockerFile 添加容器卷**

在宿主机创建文件 /mydocker/mydockerFile

```sh
# volume test
FROM centos
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
CMD echo "finished,--------success1"
CMD /bin/bash
```

docker	build	-f	/mydocker/mydockerFile	-t	loxn/centos	.	（点不能少）

对应的 宿主机目录 为 

![1565058271979](D:\笔记\note\docker\1565058271979.png)

提示：

Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied

解决办法：在挂载目录后多加一个 --privileged=true参数即可

docker -it -v /宿主机路径：/容器内目录 --privileged=true 镜像id



**3.数据卷容器**

命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为数据卷容器

先启动 dc01

docker	run	-it	--name	dc01	loxn/centos

再启动dc02，dc03

docker	run	-it	--name	dc02	--volumes-from	dc01	loxn/centos

docker	run	-it	--name	dc03	--volumes-from	dc01	loxn/centos

可以实现三个容器互相数据



## 6.dockerFile

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

构建三步骤 ===> 编写Dockerfile文件，docker build，docker run



```sh
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20190305"

CMD ["/bin/bash"]
```

**dockerFile 规则**

1：每条保留字指令都必须为大写字母且后面要跟随至少一个参数

2：指令按照从上到下，顺序执行

3：#表示注释

4：每条指令都会创建一个新的镜像层，并对镜像进行提交



**Docker执行Dockerfile的大致流程**

（1）docker从基础镜像运行一个容器

（2）执行一条指令并对容器作出修改

（3）执行类似docker commit的操作提交一个新的镜像层

（4）docker再基于刚提交的镜像运行一个新容器

（5）执行dockerfile中的下一条指令直到所有指令都执行完成



**Dockerfile保留字**

![1565088779926](D:\笔记\note\docker\1565088779926.png)



CMD 和 ENTRYPOINT 的区别

CMD：

1. Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换

ENTRYPOINT ：

1. docker run 之后的参数会被当做参数传递给 ENTRYPOINT，之后形成新的命令组合



自定义tomcat

```sh
FROM         centos
MAINTAINER    zzyy
#把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
COPY c.txt /usr/local/cincontainer.txt
#把java与tomcat添加到容器中
ADD jdk-8u171-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.8.tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_171
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.8
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.8
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE  8080
#启动时运行tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.8/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.8/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/bin/logs/catalina.out
```

启动tomcat

```sh
docker run -d -p 8080:8080 --name yfb -v /root/volumes/tomcat/yfb/webapp:/usr/local/tomcat/webapps -v /root/volumes/tomcat/yfb/logs:/usr/local/tomcat/logs --privileged=true tomcat
```

启动mysql

```sh
docker run -d -p 3306:3306 --name mysql -v /root/volumes/mysql/conf:/etc/mysql/conf.d -v /root/volumes/mysql/logs:/logs -v /root/volumes/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

启动redis

```sh
docker run -p 9379:6379 -v /root/volumes/redis/data:/data -v /root/volumes/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf -d redis redis-server /usr/local/etc/redis/redis.conf --appendonly yes
```

## 7.push 





























