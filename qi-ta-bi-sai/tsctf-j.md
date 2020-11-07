# TSCTF-J

* ## Web方向

### 0x01 easyWEB

打开页面发现就是一个输入框，输入了几次都是error

查看源代码发现一个叫submit.js的文件，内部有一个叫做BubbleGVM的函数，初步推测与检测输入有关

进入submit.js查看源代码，发现了一段代码

![](/assets/import.png)



