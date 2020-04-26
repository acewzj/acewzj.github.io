---
title: Select/Poll/Epoll
tags:
  - C
  - Select/Poll/Epoll
  - Net
categories:
  - Net
date: 2020-04-26 18:53:20
---

这篇文章记述了关于Select/Poll/Epoll的知识点。

<!--more-->

## Select

文件描述符的监视范围与第一个参数有关，实际上，select函数通过第一个参数来传递监视对象文件描述符的数量。因此，需要得到注册在fd_set变量中的文件描述符数量。但是每次新建文件描述符时，其值都会加1，故只需要将最大的文件描述符值加1再传递到select函数即可。加1是因为文件描述符的值从0开始。

```c
if((fd_num = select(fd_max+1,&cpy_reads,0,0,&timeout)) == -1)
    
for(int i = 0;i<fd_max+1;i++)
        {
            if(FD_ISSET(i,&cpy_reads))
            {
                if(i == servfd)
                {
                    clntfd = 
                     accept(i,(struct sockaddr*)&servaddr,sizeof(clntaddr));
                    FD_SET(clntfd,&reads);//reset 1
                    if(fd_max < clntfd)
                        fd_max = clntfd;//last fd occur
                    printf("connected! client :%d\n",clntfd);
                }
                else
                {
                    str_len = read(i,buf,BUF_SIZE);
                    if(str_len == 0)
                    {
                        FD_CLR(i,&reads);
                        close(i);
                        printf("closed client:%d \n",i);
                    }
                    else
                    {
                        write(i,buf,BUF_SIZE);
                    }                   
                    
                }
```

```
socklen_t clntsocklen = sizeof(clntaddr);
                    clntfd = accept(i,(struct sockaddr*)&servaddr,&clntsocklen);
```

请注意这两个代码段里面关于accept函数最后一个参数，它要求的是

`int accept(int __fd, struct sockaddr *__restrict__ __addr, socklen_t *__restrict__ __addr_len)`

其实是一个指向一个长度整数的地址。先开始我给传的是sizeof(clntaddr),但是调试时在这里返回-1.

## Epoll

#### 数字电路里的边沿触发和电平触发

边沿触发包括上升边沿触发和下降边沿触发，边沿触发检测的是电平变化，高电平转低电平或低电平转高电平时，触发一次中断。

边沿触发通过D触发器来锁存中断信号，即：若CPU来不及响应中断，外部中断信号撤消后，由于D触发器的记忆作用，消失的中断信号仍然有效，直到中断被响应并进入中断ISR，记忆的中断信号才会由硬件自动清除。

-----

电平触发分为高电平触发和低电平触发；电平触发需要手动清除中断信号

电平触发根据硬件设计的不同，分为即时触发和信号锁存触发；

（1）即时的电平触发，当外部中断信号撤消时，中断申请信号随之消失。如果在外部中断信号申请期间，CPU来不及响应此中断，那么有可能这次中断申请就漏掉了。

即时的电平触发是一个时间段，需要一直触发中断的，就用电平触发。比如高电平触发，只要检测到是高电平就触发中断。

（2）信号锁存的电平触发，当检测到高电平或低电平信号，该触发信号也会被锁存，类似于边沿触发，但是触发信号需要进行手动清除；（注意前面的边沿触发会由硬件自动清除）

 

3、边沿触发及电平触发的区别

如果是采用边沿检测外部中断，检测到电平变化会中断，但是如果中断检测口一直保持某一电平，则无法产生下次中断，需要等下次检测到电平变化才会中断。中断得到响应后由硬件自动清除。

如果是采用电平检测外部中断，检测到低/高电平会中断，但是如果中断检测口一直保持低电平，中断处理完成后，会继续产生下次中断，需要检测到高电平才会停止中断产生。中断得到响应后由硬件手动清除。

![image-20200328232423802](https://i.loli.net/2020/03/30/e7WKMlwZguRIyE4.png)



![image-20200328232049698](https://i.loli.net/2020/03/30/4FkcyDpC9ZxMiTu.png)



`epoll_create(int size)`

这个函数创建文件描述符，它创建的fd主要是用来区分epoll例程；size参数已废弃





## ![image-20200420224546779](https://i.loli.net/2020/04/20/n3NkFtTXyU5DzRO.png)

epoll:产生的事件数；及消息；使用注意

epoll 全称 eventpoll，是 linux 内核实现IO多路复用（IO multiplexing）的一个实现

epoll 监听的fd(File Descriptor)集合是常驻内核的,它有3个系统调用(*epoll_create*, *epoll_wait*, epoll_ctl)通过 *epoll_wait* 可以多次监听同一个 fd 集合，只返回可读写那部分

select 只有一个系统调用，每次要监听都要将其从用户态传到内核，有事件时返回整个集合。

从性能上看，如果 fd 集合很大，用户态和内核态之间数据复制的花销是很大的，所以 select 一般限制 fd 集合最大1024。

从使用上看，epoll 返回的是可用的 fd 子集，select 返回的是全部，哪些可用需要用户遍历判断。

尽管如此，epoll 的性能并不必然比 select 高，对于 fd 数量较少并且 fd IO 都非常繁忙的情况 select 在性能上有优势。

