---
title: （CVE-2019-0221）Apache Tomcat SSI printenv指令中的XSS
id: zhfly3201
---

# （CVE-2019-0221）Apache Tomcat SSI printenv指令中的XSS

## 一、漏洞简介

Apache Tomcat在其SSI实现中存在漏洞，可用于实现跨站点脚本（XSS）。只有在启用SSI并使用“printenv”指令的情况下，才能利用此漏洞。

利用条件

*   必须在Apache Tomcat中启用SSI支持 - 全局或特定Web应用程序。默认情况下不启用。
*   Web应用程序中必须存在具有“printenv”SSI指令的文件(通常为“.shtml”)。
*   攻击者必须能够访问该文件。

## 二、漏洞影响

Apache Tomcat 9.0.0.M1版本至9.0.0.17版本、

Apache Tomcat 8.5.0版本至8.5.39版本

Apache Tomcat 7.0.0版本至7.0.93版本

## 三、复现过程

### 漏洞分析

服务器端包含(SSI)是一些Web服务器中使用的一种简单的脚本语言，用于实现包括文件、变量的值回显和显示有关文件的基本信息等功能。这些只是针对于SSI特定的环境变量，它们要么由用户设置，要么包含关于传入HTTP请求的信息。（请参阅此处的[完整列表](https://tomcat.apache.org/tomcat-9.0-doc/ssi-howto.html#Variables)）。
“echo”指令打印出单个变量的值，而“printenv”指令打印出所有变量的值。这两个指令都输出HTML。Apache Tomcat对于使用“echo”指令时正确地转义了XSS值，但对于“printenv”指令则没有。因此，如果应用程序使用这个指令，攻击者可以注入恶意输入，从而导致XSS。
比较“echo”参数中正确转义输出的[代码](https://github.com/apache/tomcat/blob/master/java/org/apache/catalina/ssi/SSIEcho.java)：

![image](../img/548b2f5b07bbc328c78f346c73c834fb.png)

与未对输出进行正确转义的“printenv”参数的代码相比：

![image](../img/2feed46d41d0142bb24ad771dd4327da.png)

修复方法是添加如下[提交](https://github.com/apache/tomcat/commit/15fcd16)中所示的编码：

![image](../img/4cdc063aaff09e6a26dcdc3f1b6d0a28.png)

### 漏洞复现

*   1.在Windows中安装Java运行时环境(JRE)。
*   2.下载有漏洞的Tomcat版本并解压。
*   3.在第19行修改conf\context.xml文件，获得上下文权限（(这也可以在单个应用程序上执行，而不是全局执行）

```
Context privileged =“true”> 
```

*   4.根据这里的[指令](https://tomcat.apache.org/tomcat-9.0-doc/ssi-howto.html)修改conf\web.xml以启用SSI servlet(这也可以在单独的应用程序上完成，也可以是全局的)。
*   5.将以下代码放在“webapps / ROOT / ssi / printenv.shtml”中：

```
<html><head><title></title><body>
Echo test: <br /><br />
Printenv test: 
</body></html> 
```

*   6通过以下命令运行Tomcat：

```
cd bin
catalina run 
```

*   7.利用以下URL来触发XSS(可能需要使用Firefox)。观察正确转义的“echo”指令与无法正确转义的“ printenv ”指令之间的区别

```
http://www.0-sec.org:8080/ssi/printenv.shtml?%3Cbr/%3E%3Cbr/%3E%3Ch1%3EXSS%3C/h1%3E%3Cbr/%3E%3Cbr/%3E `[http://www.0-sec.org:8080/printenv.shtml?<script>alert('xss')</script>](http://www.0-sec.org:8080/printenv.shtml?%3Cscript%3Ealert(%27xss%27)%3C/script%3E)` 
```

![image](../img/f8da50e448e4091c158acd1fb68ccacf.png)

![image](../img/64c26c122a00a125f658f54612b0fe5c.png)

## 参考链接

> https://xz.aliyun.com/t/5310