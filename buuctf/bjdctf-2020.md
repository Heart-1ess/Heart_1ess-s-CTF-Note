# 0x01 Mark Loves Cat

关键词：Git泄露，变量覆盖

首先拿到网页发现无从下手，链接全部指向自身，用dirsearch扫描后发现Git泄露漏洞![](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd2.png)用GitHack下载源代码，得到index.php即主页源码![](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd3.png)通过代码审计得知index.php的重点在于如下字段![](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd4.png)

其中`$$x`与`$($x)`意义相同，因而我们整理出三个思路：

1、返回yds，并让yds=flag

2、返回is，让is=flag

3、全部绕过判断，直接输出flag

通过分析语义得知，首先输入要以a=b的形式输入，将$x与$y分别赋值为a和b，并且采用GET方式的情况下有`$($a)=$($b)`,采用POST方式的情况下有`$($a)=$b`。而后逐个判断：若输入flag并且flag===x并且x!=='flag'的时候输出handsome退出，并非所求解。若未传入flag则输出yds，若传入flag并且flag='flag'则输出is，因而最终有如下两种思路

1、若返回yds，让yds=flag只需通过GET方法传入yds=flag的语句即可进入第二条件，输出yds得到flag

2、若返回is则用GET方法传入is=flag并且flag=flag，如此可进入第三条件，输出is得到flag

值得注意的是，由于第二条件和第三条件同时不满足的情况下flag会被修改，导致最后绕过判断后无法输出正确的flag，从而第三思路不可行

最终采用GET方法传入得到flag


# 0x02 the_mystery_of_ip
关键词：SSTI，XFF伪造ip

首先拿到网页发现有个flag，进去发现能显示我们的ip地址，进入hint界面在注释中发现重点在于ip，于是我们尝试采用XFF伪造ip

![image-20210202115543302](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd5.png)

完成伪造后发现由于php代码在html解析下无法执行，因而转为尝试SSTI（实战中已经懵逼了，看wp才知道是SSTI，果然还是太菜了TAT）

![image-20210202115939427](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd6.png)

SSTI可行，进而拿到flag

![image-20210202120045487](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd7.png)

经验教训：对于每一个输出点要尽可能尝试所有方法

# 0x03 ZJCTF,不过如此

关键词：php伪协议、preg_replace的/e模式漏洞、RCE

进入网页后发现文件包含，但对于flag有过滤

![image-20210202121225470](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd8.png)

text传值采用`php://input`的伪协议，而file按照提示中的输入next.php

![image-20210202122944761](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd9.png)

成功进入next.php，下面通过`php://filter`伪协议尝试获取next.php的源代码，经过base64解码后得到：

```
<?php
$id = $_GET['id'];
$_SESSION['id'] = $id;

function complex($re, $str) {
    return preg_replace(
        '/(' . $re . ')/ei',
        'strtolower("\\1")',
        $str
    );
}


foreach($_GET as $re => $str) {
    echo complex($re, $str). "\n";
}

function getFlag(){
	@eval($_GET['cmd']);
}
```

其中在eval函数中我们看到了可以RCE的地方，下面将尝试通过cmd变量进行RCE

漏洞点在于`preg_replace`函数的`/e`模式，原句语法为执行对于$str中的内容，执行eval(strtolower("\\1"))而后输出

官方Payload：

```
/?.*={${phpinfo()}}    //phpinfo()可替换为任意想要执行的代码
\S*=${phpinfo()}
```

打入payload后，原句变为：

```
return preg_replace(
        '/(\S*)/ei',
        'strtolower("\\1")',
        {${phpinfo()}}
    );
```

即匹配所有字符，从而执行strtolower和后面的指令，即执行phpinfo，返回phpinfo的网页页面

利用该漏洞，将phpinfo替换为next.php内部的getflag()函数即可实现RCE

最终操作cmd变量获取flag



