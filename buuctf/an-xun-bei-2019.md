* # 0x01 easy\_web

打开网页发现增加了两个get类型的变量，其中image变量名称怀疑为base64加密后的数据

cmd变量为空，初步猜测为输入点![](/assets/axb1.png)经过两次base64解码后发现为十六进制字符串，翻译后得

![](/assets/abx2.png)

更改img变量内容发现左上角图片消失，判断可以采用此方法泄露源代码，此时将index.php按顺序加密后作为image变量输入

