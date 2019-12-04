# Docker常用命令

## 镜像相关命令

**1、搜索镜像**

可使用 docker search命令搜索存放在 Docker Hub中的镜像。执行该命令后， Docker就会在Docker Hub中搜索含有 java这个关键词的镜像仓库。

\# docker search java

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.gif)

以上列表包含五列，含义如下：

\- NAME:镜像仓库名称。

\- DESCRIPTION:镜像仓库描述。

\- STARS：镜像仓库收藏数，表示该镜像仓库的受欢迎程度，类似于 GitHub的 stars0

\- OFFICAL:表示是否为官方仓库，该列标记为[0K]的镜像均由各软件的官方项目组创建和维护。

\- AUTOMATED：表示是否是自动构建的镜像仓库。

 

**2、下载镜像**

使用命令docker pull命令即可从 Docker Registry上下载镜像，执行该命令后，Docker会从 Docker Hub中的 java仓库下载最新版本的 Java镜像。如果要下载指定版本则在java后面加冒号指定版本，例如：docker pull java:8

\# docker pull java:8

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image004.gif)

 

**3、列出镜像**

使用 docker images命令即可列出已下载的镜像

\# docker images

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image006.gif)

以上列表含义如下

\- REPOSITORY：镜像所属仓库名称。

\- TAG:镜像标签。默认是 latest,表示最新。

\- IMAGE ID：镜像 ID，表示镜像唯一标识。

\- CREATED：镜像创建时间。

\- SIZE: 镜像大小。

 

**4、删除本地镜像**

使用 docker rmi命令即可删除指定镜像

\# docker rmi java

 

## 容器相关命令 

**1、新建并启动容器**

使用以下docker run命令即可新建并启动一个容器，该命令是最常用的命令，它有很多选项，下面将列举一些常用的选项。

-d选项：表示后台运行

-P选项：随机端口映射

-p选项：指定端口映射，有以下四种格式。

-- ip:hostPort:containerPort 

-- ip::containerPort

-- hostPort:containerPort 

-- containerPort

--net选项：指定网络模式，该选项有以下可选参数：

--net=bridge:**默认选项**，表示连接到默认的网桥。

--net=host:容器使用宿主机的网络。

--net=container:NAME-or-ID：告诉 Docker让新建的容器使用已有容器的网络配置。

--net=none：不配置该容器的网络，用户可自定义网络配置。

\# docker run -d -p 91:80 nginx

这样就能启动一个 Nginx容器。在本例中，为 docker run添加了两个参数，含义如下：

-d 后台运行

-p 宿主机端口:容器端口 #开放容器端口到宿主机端口

访问 http://Docker宿主机 IP:91/，将会看到nginx的主界面如下：

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image008.gif)

需要注意的是，使用 docker run命令创建容器时，会先检查本地是否存在指定镜像。如果本地不存在该名称的镜像， Docker就会自动从 Docker Hub下载镜像并启动一个 Docker容器。

 

**2、列出容器**

用 docker ps命令即可列出运行中的容器

\# docker ps

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image010.gif)

如需列出所有容器（包括已停止的容器），可使用-a参数。该列表包含了7列，含义如下

\- CONTAINER_ID：表示容器 ID。

\- IMAGE:表示镜像名称。

\- COMMAND：表示启动容器时运行的命令。

\- CREATED：表示容器的创建时间。

\- STATUS：表示容器运行的状态。UP表示运行中， Exited表示已停止。 

\- PORTS:表示容器对外的端口号。

\- NAMES:表示容器名称。该名称默认由 Docker自动生成，也可使用 docker run命令的--name选项自行指定。

 

**3、停止容器**

使用 docker stop命令，即可停止容器

\# docker stop f0b1c8ab3633

其中f0b1c8ab3633是容器 ID,当然也可使用 docker stop容器名称来停止指定容器

 

**4、强制停止容器**

可使用 docker kill命令发送 SIGKILL信号来强制停止容器

\# docker kill f0b1c8ab3633

 

**5、启动已停止的容器**

使用docker run命令，即可**新建**并启动一个容器。对于已停止的容器，可使用 docker start命令来**启动**

\# docker start f0b1c8ab3633

 

**6、查看容器所有信息**

\# docker inspect f0b1c8ab3633

 

**7、查看容器日志**

\# docker container logs f0b1c8ab3633

 

**8、查看容器里的进程**

\# docker top f0b1c8ab3633

 

**9、进入容器**

使用docker container exec命令用于进入一个正在运行的docker容器。如果docker run命令运行容器的时候，没有使用-it参数，就要用这个命令进入容器。一旦进入了容器，就可以在容器的 Shell 执行命令了

\# docker container exec -it f0b1c8ab3633 /bin/bash

 

**10、删除容器**

使用 docker rm命令即可删除指定容器

\# docker rm f0b1c8ab3633

该命令只能删除**已停止**的容器，如需删除正在运行的容器，可使用-f参数

