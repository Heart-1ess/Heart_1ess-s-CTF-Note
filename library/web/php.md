# PHP

* ## 基础

Robots.txt 查看robots协议规定的网站信息（对网站本身并无实质性影响）

备份源码泄露：www.zip/www.tar.gz/index.php.bak

弱口令爆破：burp抓包后更改爆破点，加载字典进行爆破

X-Forward-For / Referer 伪造 --&gt; X-Forward-For 为最初发起请求的客户端的ip地址，Referer 为当前请求页面的来源页面地址

php弱类型对比（==），不涉及数据转换 --&gt; 0e123==0e456 （典型：md5碰撞）

HTML注入：

```
127.0.0.1;var_dump(scandir('/'));
127.0.0.1;file_get_contents(filename);
```



