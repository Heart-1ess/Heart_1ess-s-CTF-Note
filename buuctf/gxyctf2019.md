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
scandir(’.’):扫描当前目录
localeconv() 函数返回一包含本地数字及货币格式信息的数组。而数组第一项就是.
pos(),current():返回数组第一个值

//数组操作函数：
1.end():数组指针指向最后一位
2.next(): 数组指针指向下一位
3.array_reverse(): 将数组颠倒
4.array_rand(): 随机返回数组的键名
5.array_flip()：交换数组的键和值

//读取文件函数
1.file_get_content() :因为et被ban，所以不能使用
2.readfile()
3.highlight_file()
4.show_source()

session_start(): 告诉PHP使用session;
session_id(): 获取到当前的session_id值；
手动设置cookie中PHPSESSID=flag.php；
```



