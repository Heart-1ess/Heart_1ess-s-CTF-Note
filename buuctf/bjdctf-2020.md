* # 0x01 Mark Loves Cat

关键词：Git泄露，变量覆盖

首先拿到网页发现无从下手，链接全部指向自身，用dirsearch扫描后发现Git泄露漏洞![](/assets/bjd2.png)用GitHack下载源代码，得到index.php即主页源码![](/assets/bjd3.png)通过代码审计得知index.php的重点在于如下字段![](/assets/bjd4.png)

其中`$$x`与`$($x)`意义相同，因而我们整理出三个思路：

1、返回yds，并让yds=flag

2、返回is，让is=flag

3、全部绕过判断，直接输出flag

通过分析语义得知，首先输入要以a=b的形式输入，将$x与$y分别赋值为a和b，并且采用GET方式的情况下有`$($a)=$($b)`,采用POST方式的情况下有`$($a)=$b`。而后逐个判断：若输入flag并且flag===x并且x!=='flag'的时候输出handsome退出，并非所求解。若未传入flag则输出yds，若传入flag并且flag='flag'则输出is，因而最终有如下两种思路

1、若返回yds，让yds=flag只需通过GET方法传入yds=flag的语句即可进入第二条件，输出yds得到flag

2、若返回is则用GET方法传入is=flag并且flag=flag，如此可进入第三条件，输出is得到flag

值得注意的是，由于第二条件和第三条件同时不满足的情况下flag会被修改，导致最后绕过判断后无法输出正确的flag，从而第三思路不可行

最终采用GET方法传入得到flag

