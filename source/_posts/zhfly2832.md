---
title: "（CVE-2016-3088）Aria2 任意文件写入漏洞"
id: zhfly2832
---

# （CVE-2016-3088）Aria2 任意文件写入漏洞

## 一、漏洞简介

Aria2是一个命令行下轻量级、多协议、多来源的下载工具（支持 HTTP/HTTPS、FTP、BitTorrent、Metalink），内建XML-RPC和JSON-RPC接口。在有权限的情况下，我们可以使用RPC接口来操作aria2来下载文件，将文件下载至任意目录，造成一个任意文件写入漏洞。

## 二、漏洞影响

Aria2 < 1.18.8

## 三、复现过程

因为rpc通信需要使用json或者xml，不太方便，所以我们可以借助第三方UI来和目标通信，如 http://binux.github.io/yaaw/demo/ 。

> https://github.com/ianxtianxt/yaaw 源码在这里，也可自行搭建。

打开yaaw，点击配置按钮，填入运行aria2的目标域名：`http://your-ip:6800/jsonrpc`:

![image](../img/c1c3b952a2ff288e9d486fea45643c89.png)

然后点击Add，增加一个新的下载任务。在Dir的位置填写下载至的目录，File Name处填写文件名。比如，我们通过写入一个crond任务来反弹shell：

![image](../img/0a901be479c83f9f4dc67a65a8f7e396.png)

这时候，arai2会将恶意文件（我指定的URL）下载到/etc/cron.d/目录下，文件名为shell。而在debian中，/etc/cron.d目录下的所有文件将被作为计划任务配置文件（类似crontab）读取，等待一分钟不到即成功反弹shell：

![image](../img/c0ffc45bd6bcdd2f6b288f54349c75fa.png)

> 如果反弹不成功，注意crontab文件的格式，以及换行符必须是`\n`，且文件结尾需要有一个换行符。