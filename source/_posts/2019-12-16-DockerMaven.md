---
title:  "Docker"
date:   2019-12-16 10:16:18 +0800
categories:
- java
tags: docker
---

这篇文章主要记述了docker容器的一些知识。


<!--more-->






# docker

![](https://i.loli.net/2019/12/14/2IHTljsdOekP1VC.png)

–name 参数，这是给容器实例起了个名字方便后续的守护进程调用，如果不加这个参数会随机产生一个名字。如果加名字切记如果多次run会提示名字冲突，需要先删除之前run的实例。

- 通过 **docker ps -a** 命令查看已经在运行的容器，然后使用容器 ID 进入容器。

- 使用“docker attach”命令进入container（容器）有一个缺点，那就是每次从container中退出到前台时，container也跟着退出了。要想退出container时，让container仍然在后台运行着，可以使用“docker exec -it”命令。每次使用这个命令进入container，当退出container后，container仍然在后台运行，命令使用方法如下：

```
docker exec -it goofy_almeida /bin/bash
goofy_almeida：要启动的container的名称
```

/bin/bash：在container中启动一个bash shell

这样输入“exit”或者按键“Ctrl + C”退出container时，这个container仍然在后台运行，通过：`docker ps`
就可以查找到

- elasticsearch 经常启动后退出，exit返回137，后来重启服务器就好了，推测可能跟虚拟内存有关，我的内存才2G。
- docker: Error response from daemon: OCI runtime create failed: container_linux.go:346**导致**kibana安装失败

![](https://i.loli.net/2020/03/29/PmGn7tiMgNFvH3S.png)

这里有误应为`docker container cp ./mall.sql mysql:/`

#### 让Docker支持http上传镜像

```python
echo '{ "insecure-registries":["47.97.25.88:5000"] }' > /etc/docker/daemon.json
```

```json
<dockerHost>http://47.97.25.88:2375</dockerHost>
```

## Docker Compose常用命令

### 构建、创建、启动相关容器：

```
# -d表示在后台运行docker-compose up -d
```

### 停止所有相关容器：

```
docker-compose stop
```

### 列出所有容器信息：

```
docker-compose ps
```

## maven

为什么我已经有一个pom.xml了，每一个模块里面还有一个pom.xml?

- project：pom.xml的顶级元素。

- groupId：指出创建这个工程的组织或团队的唯一标识。

- plugins：插件。

- artifactId：基本名称。

- packaging：类型（如JAR、WAR、EAR等等），默认是JAR，所有带有子模块的项目的packaging

  都为pom。

- version：版本号。

- modelVersion：指出POM使用哪个版本的对象模型。




## Nginx

明白了反向代理，是通过nginx服务器代理，访问不同的域名不同的端口（多个服务器域名和端口不可能重复）之间转发，比如，我想用户访问域名为www.web.cn:8888，nginx会帮我转发请求到其中一个apache什么的web服务器，如果是www.java.cn:8080，nginx就帮我转发给其他一个tomcat服务器做处理。要做到这样多站点多域名的转发，我们只要配置一下nginx代理服务器的nginx.conf文件就可以了，可以直接在里面配置，也可以多建立几个conf文件，让nginx.conf文件include包含他们，系统会自动扫描那些conf文件。这样配置后，当我们访问某个域名时候，nginx会主动转发到配置那个域名的conf文件，里面的root配置的就是服务器指向的项目运行根目录，或者配置反向代理proxy_pass 转发的域名，这样就反向代理转发到你想转发的服务器路径。

```
//部署多个应用
docker run -p 80:80 --name nginxhtml \
-v /mydata/nginx/www/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx  \
-v /mydata/nginx/conf:/etc/nginx \
-d nginx:1.10

//原始的
docker run -p 10010:80 --name nginxmall \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx  \
-v /mydata/nginx/conf:/etc/nginx \
-d nginx:1.10

//开启防火墙
firewall-cmd --zone=public --add-port=9200/tcp --permanent
firewall-cmd --reload
```

