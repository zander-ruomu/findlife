# Docker 安装

以 CentOS 系统为例

1、Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

通过 uname -r 命令查看你当前的内核版本

```bash
uname -r
```

2、使用 root 权限登录 Centos。确保 yum 包更新到最新。

```bash
yum -y update
```

3、卸载旧版本(如果安装过旧版本的话)

```bash
yum remove docker docker-common docker-selinux docker-engine
```

4、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

5、设置yum源

```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

![9-1433322312.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.gif)

6、可以查看所有仓库中所有docker版本，并选择特定版本安装

```bash
yum list docker-ce --showduplicates | sort -r
```

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image004.gif)

7、安装docker

```bash
sudo yum install docker-ce #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版18.03.1
sudo yum install <FQPN> # 例如：sudo yum install docker-ce-18.03.1.ce
```

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image006.gif)

8、启动并加入开机启动

```bash
systemctl start docker 
systemctl enable docker
```

9、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

\# docker version

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image008.gif)

 

10、卸载docker

\# yum -y remove docker-engine



**注意：需要配置镜像加速器**

\# docker search java

Error response from daemon: Get https://index.docker.io/v1/search?q=java: read tcp 52.200.132.201:443: i/o timeout

我们可以借助阿里云的镜像加速器，登录阿里云(https://cr.console.aliyun.com/#/accelerator)

可以看到镜像加速地址如下图：

![clipboard.png](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image010.gif)

```bash
cd /etc/docker
```

查看有没有 daemon.json。这是docker默认的配置文件。

如果没有新建，如果有，则修改。

```bash
vim daemon.json
```

i

{

 "registry-mirrors": ["https://m9r2r2uj.mirror.aliyuncs.com"]

}

wq! 保存退出。

重启docker服务

```bash
service docker restart
```

成功！

