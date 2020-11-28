## 0x01 禁止套娃

用dirsearch扫描发现是git泄露，采用GitHack下载泄露的Git发现是index.php![](/assets/GXY1.png)打开Index.php进行源码审计，发现过滤：

```
<?php
include "flag.php";
echo "flag在哪里呢？<br>";
if(isset($_GET['exp'])){
    if (!preg_match('/data:\/\/|filter:\/\/|php:\/\/|phar:\/\//i', $_GET['exp'])) {
        if(';' === preg_replace('/[a-z,_]+\((?R)?\)/', NULL, $_GET['exp'])) {
            if (!preg_match('/et|na|info|dec|bin|hex|oct|pi|log/i', $_GET['exp'])) {
                // echo $_GET['exp'];
                @eval($_GET['exp']);
            }
            else{
                die("还差一点哦！");
            }
        }
        else{
            die("再好好想想！");
        }
    }
    else{
        die("还想读flag，臭弟弟！");
    }
}
// highlight_file(__FILE__);
?>
```

* 过滤伪协议

* \(?R\)引用当前表达式，后面加了?递归调用。只能匹配通过无参数的函数。

* 过滤了et\|na\|info\|dec\|bin\|hex\|oct\|pi\|log关键字。

知识点：

```

```



