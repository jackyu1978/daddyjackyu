+++
title = "http code选择"
description = ""
keywords = ["",""]
categories = [""]
date = "2016-01-01T20:58:19+08:00"
url = "/2016/01/01/http-code-select/"

+++

在web开发过程中，当程序处理过程中，不可避免的面临http响应代码的选择<!--more-->，大家最了解的可能就是成功时返回的：200 OK，但还有很多错误码，往往对应着程序出错逻辑的处理。见到参考资料中的http code选择的文章，觉得其中的错误码选择逻辑图片对我们的程序逻辑处理，及程序安全性逻辑方面都有很好的参考作用，特推荐给大家。希望对大家的web程序开发能有所帮助。

# HTTP CODE分类 #

![](http://i.imgur.com/oQhQlbR.png)

# 2XX/3XX #
![](http://i.imgur.com/0Bmax1D.png)

# 4XX #
![](http://i.imgur.com/VULNqcj.png)

# 5XX #
![](http://i.imgur.com/ELXtruL.png)


# HTTP CODE列表 #

## 1×× Informational ##

- 100 Continue
- 101 Switching Protocols
- 102 Processing

## 2×× Success ##

- 200 OK
- 201 Created
- 202 Accepted
- 203 Non-authoritative Information
- 204 No Content
- 205 Reset Content
- 206 Partial Content
- 207 Multi-Status
- 208 Already Reported
- 226 IM Used

## 3×× Redirection ##

- 300 Multiple Choices
- 301 Moved Permanently
- 302 Found
- 303 See Other
- 304 Not Modified
- 305 Use Proxy
- 307 Temporary Redirect
- 308 Permanent Redirect

## 4×× Client Error ##

- 400 Bad Request
- 401 Unauthorized
- 402 Payment Required
- 403 Forbidden
- 404 Not Found
- 405 Method Not Allowed
- 406 Not Acceptable
- 407 Proxy Authentication Required
- 408 Request Timeout
- 409 Conflict
- 410 Gone
- 411 Length Required
- 412 Precondition Failed
- 413 Payload Too Large
- 414 Request-URI Too Long
- 415 Unsupported Media Type
- 416 Requested Range Not Satisfiable
- 417 Expectation Failed
- 418 I'm a teapot
- 421 Misdirected Request
- 422 Unprocessable Entity
- 423 Locked
- 424 Failed Dependency
- 426 Upgrade Required
- 428 Precondition Required
- 429 Too Many Requests
- 431 Request Header Fields Too Large
- 451 Unavailable For Legal Reasons
- 499 Client Closed Request

## 5×× Server Error ##

- 500 Internal Server Error
- 501 Not Implemented
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout
- 505 HTTP Version Not Supported
- 506 Variant Also Negotiates
- 507 Insufficient Storage
- 508 Loop Detected
- 510 Not Extended
- 511 Network Authentication Required
- 599 Network Connect Timeout Error


## 参考文章： ##

[http://racksburg.com/choosing-an-http-status-code/](http://racksburg.com/choosing-an-http-status-code/)

[https://httpstatuses.com/](https://httpstatuses.com/)