---
title: Emlog 越权&后台getshell
id: zhfly2915
---

# Emlog 越权&后台getshell

## 一、漏洞简介

## 二、漏洞影响

## 三、复现过程

### 代码分析

#### 安装漏洞

​ 其实一般代码审计都是从安装文件开始审计，一般安装脚本主要存在如下漏洞：

```
无验证功能，任意重装覆盖
表单不做过滤写入config.php导致getshell
$_GET['step']跳过限制步骤
漏洞文件：install.php 
```

​
​首先我们直奔第一个点能否任意重装，我们可以看到必须常量DEL_INSTALLER为1的时候才会触发删除install.php,那么我们追踪DEL_INSTALLER看看

![image](../img/b053bb7dec06cb14c3e464e5f1cd345d.png)

这里可以看到DEL_INSTALLER默认值就是0，所以一般情况下这里是可以任意重装的，我们从黑盒的测试也可以印证这一点。同时表单也做了过滤所以这里也没有后面两种情况。

![image](../img/4a7a60e5d4fb70134f83c77f22a755d7.png)

#### 越权漏洞

​
​在安装完毕后打开config.php看到两个比较奇怪的常量定义：AUTH_KEY和AUTH_COOKIE_NAME从名字来看这连个常量肯定是有一定联系的。如下图：

![image](../img/8b4131e721501fecd9e11c87720544d9.png)

我们在抓包时候发现了AUTH_COOKIE_NAME这个常量，说明这是一个cookie名。我们继续再代码中追踪这个常量。

![image](../img/4ff1f6478a93a07d2586af4a984a7bf8.png)

我们可以看到在登录验证cookie中使用了这个常量，我们追踪一下cookie值是如何构造的

![image](../img/547d38dd2e23899732a32f9e78309c38.png)

这里可以看到，调用了emHash这个类方法，并且这里使用了AUTH_KEY这个常量，这里说明这两个常量是有联系的。我们在继续追踪hash_hmac()到底使用key做了什么。

![image](../img/8aea7c6e96ca034d9974f91d3266058c.png)

这里对传入的$key也就是AUTH_KEY进行了md5加密到一个二进制字符串中而后分割为64个字节与一个字符*64次的字符串进行异或最后得到两个字符串$ipad,$opad最后再将他们打包拼接用md5加密返回给上级调用。我们再回到上级。

![image](../img/ead3cee1b7af9400c92e9bec200ef2bb.png)

我们知道generateAuthCookie方法中的 $key、$hash是由AUTH_KEY加密而成。最终的cookie是由$user_login 、 $expiration $hash拼接而成，而$expiration是cookie的生存时间，$user_login是用户名。这里可以得知$key、$user_login、$expiration都是固定的那么只要知道AUTH_KEY就有伪造cookie造成越权的可能。

​例如我们准备两个靶机，一个靶机登陆，获取这个靶机的cookie即可越权登陆另外一个靶机。前提是两个靶机的AUTH_KEY得一致。这里有点鸡肋但是还是有利用的可能。我们将两个靶机的cookie拿出来比较确实是一样的。
​

![image](../img/bc8d37d892c3cd41aa70ca257f6bf25d.png)

#### 后台getshell（一）

​
​其实上诉两个漏洞已经能够让我们进入后台了，现在的任务就是如何Getshell了。这里我随便看了一下发现后台有设置上传附件后缀的功能。
​

![image](../img/a8eb5a7800af8bf57bb1e771435e9090.png)
​
​ 但是测试发现，加入php后缀你发现会将phpt替换为X,从源码中可有很直观看到这点。
​
![image](../img/8a3183b31a93e65ce9747fb903a9a00c.png)
​
​此时我注意到了备份功能，这个后缀是保存在数据库中的，既然我不能直接将后缀写入数据库，那么我能不能通过数据备份恢复的方法写入数据库呢。
​
​ 这里直接备份所有表，下载下来然后找到写入后缀的语句，加上php,如下
​
![image](../img/079a90da2e05f5d0134b818fbb8c0976.png)
​
![image](../img/3b4bdc5557d79b735daab1b1440c1745.png)
​
​ 导入备份后发现设置中的上传后缀有php
​
![image](../img/146739facd5d2ee373151eb54bfc4654.png)
​
​ 直接在文章发表出上传PHP文件即可getshell
​
![image](../img/5302808d519640b7b4ede8c191329b77.png)

#### 后台getshell（二）

我们还发现这个cms中上传插件的地方可以上传zip，这里我们想如果在插件中插入一句话木马是不是也能getshell?我们找到插件上传的脚本，发现使用了一个叫emUnzip的函数。

![image](../img/547d38dd2e23899732a32f9e78309c38.png)

我们继续追踪这个函数，发现这个解压函数是使用ZipArchive()类来实现解压缩的。我们可以看到代码755行中获取了压缩包的内部目录/文件的名称 ,并将其分割为数组将第一个元素赋值给了$dir，我们看到switch中plugin选项，这里又将获取的第一个文件夹名称赋值给$plugin_name,使用getFromName方法获取了压缩包是否存在$dir . $plugin_name . '.php'这个文件。综上这里就是检测压缩包中文件夹里面是否存在一个与文件夹名称一致的PHP文件，最后在再压。这里也没对文件进行其他校验操作。因此我们只要再构造 文件夹名和文件名相同的内容的压缩包，同时由上方代码也可以知道，文件将会被解压到 /content/plugins/a/a.php(文件夹名称a)

![image](../img/c0ae43a0e74e222660b26006346c368e.png)

构建压缩包上传插件：

![image](../img/530973e56f55bdb2ca436dc3efa857c4.png)

## 四、参考链接

> https://xz.aliyun.com/t/6861