+++
title = "利用nginx防御DDoS攻击"
description = ""
name = "nginx-ddos"
keywords = ["nginx","DDos"]
categories = [""]
date = "2015-12-24T22:16:41+08:00"
url = "/2015/12/24/ngix-ddos/"

+++

nginx有很多特性可以用来有效防御DDoS攻击

<!--more-->

## 限制请求频率 ##
DDoS攻击的攻击者的请求频率肯定会大于真实用户请求的频率，这样才能起到攻击的目的，如实我们就可以利用nginx来限制客户端请求的频率，具体设置的阈值可以根据实际业务场景及以前的经验值来确定。
比如你可以配置nginx限制同一IP地址对login页面的请求为每次/2秒（折算成一分钟就是最大30次），超过这个阈值nginx就会阻断：

    limit_req_zone $binary_remote_addr zone=one:10m rate=30r/m;

    server {
    ...
	    location /login.html {
	        limit_req zone=one;
	    ...
	    }
    }
limit\_req\_zone指令配置一段名字为 **one** 的共享内存区域，来存储请求状态，关键字为客户端IP($binary\_remote\_addr)。**/login.html** 的location块中的limit\_req指令引用该共享内存区域。

## 限制链接数 ##

DDoS的攻击者往往发起大量的网络链接，以期起到攻击目的。所以我们可以限制每个单IP客户端的链接数，甚至可以限制每个客户端IP可以发起到网站特定区域或路径的链接数。比如：限制每个开发可以发起到**/store**区域的链接数为10：

    limit_conn_zone $binary_remote_addr zone=addr:10m;

    server {
    ...
	    location /store/ {
	        limit_conn addr 10;
	        ...
	    }
	}

limit\_conn\_zone指令配置一段名字为 **addr** 的共享内存区域，来存储请求状态，关键字为客户端IP($binary\_remote\_addr)。**/store** 的location块中的limit\_conn指令引用该共享内存区域,并限制每个IP的链接数最大为10。

> **注意：国内很多用户都是通过NAT上网，同一个局域网中的用户出口IP是一样的，所以在设置链接数限制时，要考虑这种情况，不然可能会导致服务不可用！**

## 关闭慢速链接 ##
Http慢速攻击往往也是DDoS攻击中一种常见的攻击形式，利用nginx的**client\_body\_timeout** 和 **client\_header\_timeout**指令也可以防护这方面的攻击：

    server {
    	client_body_timeout 5s;
    	client_header_timeout 5s;
    	...
	}
client\_body\_timeout指令：控制客户端在两次发送Body数据之间的间隔。

client\_header\_timeout指令：控制客户端在两次发送http头数据之间的间隔。

## 设置IP地址黑名单 ##
如果你已经确定某个客户端IP正在发动攻击，可以通过设置IP地址黑名单的方式加以阻断。

设置IP地址段：

    location / {
    deny 123.123.123.0/28;
    ...
	}


设置单个IP地址：

    location / {
    	deny 123.123.123.3;
    	deny 123.123.123.5;
    	deny 123.123.123.7;
    	...
	}


## 设置IP地址白名单 ##
如果你的网站只允许特定区域的用户访问，就可以采用设置白名单的方式加以保护：

    location / {
    	allow 192.168.1.0/24;
		deny all;  #阻断除白名单外的其他客户端访问
    	...
	}

## 利用缓存机制来减少后端服务请求 ##
我们可以配置nginx来缓存内容，以此来减少到后端服务器的请求，达到减少通讯，缓解DDoS攻击的目的，其中有两个指令比较有帮助：

1. proxy\_cache\_use\_statle:告诉nginx，在缓存过期的时候怎么处理，为防止DDoS，这里可以设置为*on*
2. proxy\_cache\_key:设置缓存的关键字，默认值（$scheme$proxy\_host$request_uri）。**注意：**如果关键字包含$query\_string变量，攻击者就可以发送一些随机query strings导致大量无用缓存信息。所以一般推荐不要加上$query\_string参数。


## 阻断请求 ##
可以配置nginx来阻断一些类型的请求：

- 特别的被攻击的URL
- 包含特别的User-Agent头的请求
- 包含特别Referer头的请求
- 包含其他特别Http内容的请求

例如：


    location /foo.php {  #阻断对/foo.php的访问
    	deny all;
	}

	location / { #阻断User-Agent头的值为foo或bar的请求
	    if ($http_user_agent ~* foo|bar) {
	        return 403;
	    }
	    ...
	}
$http\_*name*变量代指特定的请求头，如$http\_user\_agent

## 限制到后端服务器的链接 ##

nginx往往比backend服务器可以处理更大量的并发链接，在nginx上配置backend服务器的链接限制也是个不错的策略：

	upstream website {
	    server 192.168.100.1:80 max_conns=200; #最大连接数200
	    server 192.168.100.2:80 max_conns=200; 
	    queue 10 timeout=30s; #queue:所有服务器都达到最大链接数时的队列数，timeout:多长时间从队列中获取一个请求
	}

## 应对Range攻击 ##
一种攻击方式是发送一个http投中Range是很大值的请求，导致缓冲区溢出：

    GET / HTTP/1.1\r\n
    Host: stuff\r\n
    Range: bytes=0-18446744073709551615\r\n
    \r\n

应对这种攻击的其中一种方式是使用proxy\_set\_header指令设置Range为空字符串（""），这会让nginx在将请求转发到后端服务器的时候删除Range字段：

    server {
	    listen 80;
	    
	    location / {
		    proxy_set_header Range "";
		    proxy_pass http://windowsserver:80;
	    }
    }

如果你的应用需要用到byte-range,可以使用下面的方法来检查Range字段的值是否超大：

    map $http_range $saferange {
        "~\d{10,}" "";  # if it matches a string of 10 or more integers, remove it
        default $http_range;
	}

	server {
        listen 80;

        location / {
                proxy_set_header Range $saferange;
                proxy_pass http://windowsserver:80;
        }
	}

当Range的值超大的时候也可以返回444错误码，让nginx直接关闭连接：

    server {
        listen 80;

        if ($http_range ~ "\d{9,}") {
            return 444;
        }

        location / {
                proxy_pass http://windowsserver:80;
        }
	}

## 处理高负载 ##
DDoS攻击通常导致高负载。可以对nginx和操作系统网络配置方面做些优化来处理高负载问题，详细内容见下篇[nginx调优](http://www.daddyjackyu.com/2015/12/26/ngix-better/)





----------



**参考**
[https://www.nginx.com/blog/mitigating-ddos-attacks-with-nginx-and-nginx-plus/](https://www.nginx.com/blog/mitigating-ddos-attacks-with-nginx-and-nginx-plus/)