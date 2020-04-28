---
title: Config_Git
date: 2020-04-06 08:09:03
tags:
- git
- svn
---



这篇文章主要记述了Git、SVN和以及加密传输的一些知识点。

<!--more-->



## GitHub

1.先生成秘钥

ssh-keygen -t rsa -C "[xxxxx@xxxxx.com](mailto:xxxxx@xxxxx.com)"  

\# Generating public/private rsa key pair...

\# 三次回车即可生成 ssh key



1. $ git clone "SSH地址"
2. \#git pull origin master
3. \#git add .
4. \#git commit -m 'init'
5. \#git push origin master
6. git checkout -b ‘name’

## Git配置多个SSH-Key

[SSH Key](https://gitee.com/help/labels/19)

### 背景

当有多个git账号时，比如：

a. 一个gitee，用于公司内部的工作开发；
b. 一个github，用于自己进行一些开发活动；

### 解决方法

1. 生成一个公司用的SSH-Key

```
$ ssh-keygen -t rsa -C 'xxxxx@company.com' -f ~/.ssh/gitee_id_rsa
```

1. 生成一个github用的SSH-Key

```
$ ssh-keygen -t rsa -C 'xxxxx@qq.com' -f ~/.ssh/github_id_rsa
```

1. 在 ~/.ssh 目录下新建一个config文件，添加如下内容（其中Host和HostName填写git服务器的域名，IdentityFile指定私钥的路径）

```
# gitee
Host gitee.com
HostName gitee.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/gitee_id_rsa
# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/github_id_rsa
```

4.用ssh命令分别测试

```
$ ssh -T git@gitee.com
$ ssh -T git@github.com
```

##VSCODE 配置SSH

```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa-remote-ssh

$ ssh-copy-id -i ~/.ssh/id_rsa-remote-ssh root@47.97.25.88
```



------