+++
title = "nginx调优"
name = "nginx-better"
description = ""
keywords = ["nginx","DDos"]
categories = [""]
date = "2015-12-26T19:49:42+08:00"
url = "/2015/12/26/ngix-better/"

+++

nginx作为一款知名的高性能web服务器、代理服务器<!--more-->，使用nginx和linux的默认配置一般就可以达到很不错的性能。但在有些特殊情况下我们需要对nginx和系统的配置做些调整以达到更高的并发，更大的吞吐量。本篇文章就讨论下有哪些配置可以调整。

# 概述 #

由于需要调整nginx和linux的参数，所以首先需要大家对nginx和linux系统相关的配置有些基本了解，这样才能更好理解这些配置调整的目的。

另外在实际调整的时候，请遵循一个规则：

**“一次只调整一个参数，然后对比看性能有没有提升，如果没有提升请改回默认配置。”**

# 调整linux配置 #
目前linux系统（2.6+）的默认配置满足正常的大部分使用场景，但改变一些参数的设置也可以相应提高性能。

## 调整Backlog队列 ##
下面的设置可以调整linux的网络链接处理的性能，如果大量链接被挂起，可以尝试调整下面的参数：

- net.core.somaxconn:nginx可以accept的最大链接数队列。缺省值经常都较小，但nginx处理accetp的速度又很快。如果你的网站并发数大的话，可以尝试调整该参数。有时候该值太小linux kernel 日志中会有错误信息，可以增加该参数的值，直到错误消失。

**注意：**如果该值设置的大于512的时候，相应的改变nginx listen指令中backlog的值与之对应

- net.core.netdev\_max\_backlog:网络包在被CPU处理之前被网卡缓存的比率。当带宽较大时，增大该值可以提高性能。

## 调整文件描述符 ##
文件描述符是系统表示网络链接和打开文件的一种系统资源。当nginx在代理模式时，一个请求需要占用两个文件描述符，一个表示客户端的链接，一个代表到被代理服务器的网络链接。当系统处理大量链接的时候，相关的参数需要做调整：

- sys.fs.file\_mas:系统最大文件描述符限制
- nofile:用户描述符限制，在/etc/security/limits.conf文件中设置

## 零时端口 ##
nginx作为代理的时候，每个到upstream服务器的链接都要占用一个零时端口，可以调整下面的值优化：

- net.ipv4.ip\_local\_port_range:本地端口起始-结束范围，如果你发现提示没有零时端口可用，就可以增大这个范围。通常的可以设置 1024-65000
- net.ipv4.tcp\_fin\_timeout:零时端口可被重用的超时时间。缺省是60秒，通常可以缩短到30或15秒

# 调整nginx的配置 #
下面列出一些会影响nginx性能的指令和配置项

## 工作进程数 ##
nginx可以同时启动多个进程，每个进程处理一定的网络链接，你可以控制启动的工作进程数及他们处理网络链接的方式：

- worker_processes:工作进程数，缺省值1.通常一个CPU核心可以分配一个工作进程，其值可以设置成CPU核心数。如果程序中需要处理很多的磁盘I/O,可以适当调大该值。
- worker_connections:每个工作进程可以处理的最大并发链接数。缺省值是512，但是系统一般有充分的资源可以支持更大的链接数。最佳值一般依赖服务器性能和网络状况，可以多次修改测试对比。

## Keepalive设置 ##
keepalive链接可以很大程度上减少CPU和网络负载，不用频繁创建和关闭链接。nginx在客户端链接和upstream链接中都支持Keepalive选项。下面的指令是跟客户端链接有关的：

- keepalive_request:一个keepalive长链接中，客户端可以发起的请求数，缺省值是100。
- keepalive_timeout:keepalive空闲超时时间

下面的指令和upstream链接有关：

- keepalive--每个工作进程保持到上游服务器的空闲KeepAlive连接数。没有默认值。
为了启用到upstream服务器的长连接，还必须包含以下指令：

    proxy\_http\_version 1.1;  
	proxy\_set\_header Connection ""; 

## 记录访问日志 ##
记录每个请求的日志会消耗CPU和磁盘I/O,为了减少消耗可以开启访问日志记录缓存。

- 在access_log指令中添加buffer=*size*参数
- 在access_log指令中添加flush=*time*参数
- 当工作进程打开或关闭日志文件时，也会记录对应的日志，可以在access_log指令中添加off参数关闭

## Sendfile ##
操作系统的sendfile()系统调用可以零拷贝将数据从一个文件描述符拷贝到另一个描述符，这可以提高TCP数据传输。这可以通过在http、server、location上下文中添加sendfile指令让nginx使用它。因为sendfile拷贝数据不通过用户空间，所以会破坏nginx的处理连和改变内容的过滤，比如输出gzip。当同时配置了sendfile指令和改变内容指令过滤器时，nginx自动让sendfile指令失效。

## 对请求做限制 ##
可以对客户端请求做一些限制，以防止消耗过多资源。这同时可以提高性能和安全性，相关指令如下：

- limit_conn和limmit\_conn\_zone:限制客户端链接数
- limit_rate:限制对客户端的响应发送速率
- limit和limit\_req\_zone:限制请求速率
- max\_conns，upstream配置块中server指令的一个参数，设置upstream服务器的并发链接数，如果设置为0（缺省为0），则表示没限制
- queue(nginx plus):设置当所有upstream服务器都达到max\_conns限制时，队列中的最大请求数。如果没有这个参数，请求不会被放入队列中

## 缓存和压缩 ##
### 缓存 ###
开启缓存可以减小upstream server的负载，提高响应速度

### 压缩 ###
对响应进行压缩可以减小带宽消耗，但压缩会消耗CPU资源，但一般来说带宽比CPU贵。而对已一些已经压缩过的内容，如JPEG文件，就不用再压缩了。


**参考文章：**[tuning-nginx](https://www.nginx.com/blog/tuning-nginx/ "tuning-nginx")

