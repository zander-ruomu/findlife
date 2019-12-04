# SVN服务平台搭建

## 服务端部署

1）安装前装备,Linux需要安装的软件

```bash
 yum install subversion (SVN服务器) mysql-server (用于codestriker)
 			 httpd mod_dav_svn mod_perl (用于支持WEB方式管理SVN服务器)
 			 sendmail (用于配置用户提交代码后发邮件提醒)
 			 wget gcc-c++ make unzip perl* (必备软件包)
 			 ntsysv vim-enhanced (可选)
```

2）基本的SVN服务器配置

新建一个目录用于存储SVN所有文件

```bash
mkdir /home/svn
```

新建一个版本仓库

```bash
svnadmin create /home/svn/project
```

初始化版本仓库中的目录

```bash
cd /tmp
mkdir project project/server project/client project/test #建立临时目录
svn import project/ file:///home/svn/project/ -m*** #初始化SVN目录
rm -rf project (删除临时建立的目录)
```

3) 添加用户

1. 在/home/svn/project/conf/passwd文件中添加一个形式为”username=password”的条目就可以了，如下:

<img src="file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg" alt="img" style="zoom: 150%;" />

2. 修改用户访问策略，修改配置文件/home/svn/project/conf/authz，如下:

<img src="file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg" alt="img" style="zoom:150%;" />

3. 修改svnserve.conf文件，让用户和策略配置生效，如下：

   <img src="file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg" alt="img" style="zoom:150%;" />

4) 启动服务器

```bash
svnserve -d -r /home/svn
```

5) 测试服务器

```bash
 svn co svn://192.168.60.10/project #此时访问的是project目录，应该以dengzhe的身份登陆
```

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg)

6) 若登陆出错，服务器会记录登陆信息，解决办法，如下：

Linux报错：

<img src="file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg" alt="img" style="zoom:150%;" />

解决方法，删除访问日志文件：rm -rf ~/.subversion/auth/



## 客户端配置

1） 下载代码路径

![image-20191204094035964](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191204094035964.png)

![image-20191204094204432](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20191204094204432.png)



2） Windows权限报错解决办法：

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg)