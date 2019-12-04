# **Docker部署微服务**

## 使用Dockerfile构建Docker镜像

Dockerfile是一个文本文件，其中包含了若干条指令，指令描述了构建镜像的细节

先来编写一个最简单的Dockerfile，以Nginx镜像为例，来编写一个Dockerfile修改Nginx镜像的首页

1、新建文件夹/app，在app目录下新建一个名为Dockerfile的文件，在里面增加如下内容：

FROM nginx

RUN echo '<h1>This is Tuling Nginx!!!</h1>' > /usr/share/nginx/html/index.html

该Dockerfile非常简单，其中的 FORM、 RUN都是 Dockerfile的指令。 FROM指令用于指定基础镜像， RUN指令用于执行命令。

2、在Dockerfile所在路径执行以下命令构建镜像： 

\# docker build -t nginx:tuling .

其中，-t指定镜像名字，命令最后的点（.）表示Dockerfile文件所在路径

3、执行以下命令，即可使用该镜像启动一个 Docker容器

\# docker run -d -p 92:80 nginx:tuling

4、访问 http://Docker宿主机IP:92/，可看到下图所示界面

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.gif)

## Dockerfile常用指令

| 命令             | 用途                                                         |
| ---------------- | ------------------------------------------------------------ |
| FROM             | 基础镜像文件                                                 |
| RUN              | 构建镜像阶段执行命令                                         |
| ADD <src> <dest> | 添加文件，从src目录复制文件到容器的dest，其中 src可以是  Dockerfile所在目录的相对路径，也可以是一个 URL,还可以是一个压缩包 |
| COPY             | 拷贝文件，和ADD命令类似，但不支持URL和压缩包                 |
| CMD              | 容器启动后执行命令                                           |
| EXPOSE           | 声明容器在运行时对外提供的服务端口                           |
| WORKDIR          | 指定容器工作路径                                             |
| ENV              | 指定环境变量                                                 |
| ENTRYPINT        | 容器入口， ENTRYPOINT和  CMD指令的目的一样，都是指定 Docker容器启动时执行的命令，可多次设置，但只有最后一个有效。 |
| USER             | 该指令用于设置启动镜像时的用户或者 UID,写在该指令后的 RUN、 CMD以及  ENTRYPOINT指令都将使用该用户执行命令。 |
| VOLUME           | 指定挂载点，该指令使容器中的一个目录具有持久化存储的功能，该目录可被容器本身使用，也可共享给其他容器。当容器中的应用有持久化数据的需求时可以在 Dockerfile中使用该指令。格式为：  VOLUME["/data"]。 |

注意：RUN命令在 image 文件的构建阶段执行，执行结果都会打包进入 image 文件；CMD命令则是在容器启动后执行。另外，一个 Dockerfile 可以包含多个RUN命令，但是只能有一个CMD命令。

注意，指定了CMD命令以后，docker container run命令就不能附加命令了（比如前面的/bin/bash），否则它会覆盖CMD命令。

 

## 使用Dockerfile构建微服务镜像

以项目**05-ms-eureka-server**为例，将该微服务的可运行jar包构建成docker镜像

1、将jar包上传linux服务器/app/eureka目录，在jar包所在目录创建名为Dockerfile的文件

2、在Dockerfile中添加以下内容

\# 基于哪个镜像

From java:8

\# 将本地文件夹挂载到当前容器

VOLUME /tmp

\# 复制文件到容器

ADD microservice-eureka-server-0.0.1-SNAPSHOT.jar /app.jar

\# 声明需要暴露的端口

EXPOSE 8761

\# 配置容器启动后执行的命令

ENTRYPOINT ["java","-jar","/app.jar"]

3、使用docker build命令构建镜像

\# docker build -t microservice-eureka-server:0.0.1 .

\# 格式： docker build -t 镜像名称:标签 Dockerfile的相对位置

在这里，使用-t选项指定了镜像的标签。执行该命令后，终端将会输出如下的内容

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image004.gif)

4、启动镜像，加-d可在后台启动

\# docker run -p 8761:8761 microservice-eureka-server:0.0.1

5、访问http://Docker宿主机IP:8761/，可正常显示Eureka Server首页

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image006.gif)

 