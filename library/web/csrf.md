# CSRF&SSRF（跨站请求伪造和服务器端请求伪造）
### CSRF：利用用户身份去做未经过用户授权的行为
### SSRF：伪造用户身份欺骗服务器去做一些行为
* ## 核心：伪造身份

* ### 经由：伪造签名、伪造ip地址等

* #### 方法：session伪造、JWT伪造、哈希长度扩展攻击、

* ## JWT伪造：
  通过伪造jwt达成伪造身份的目的

* ## 哈希长度扩展攻击：

* md5原理：<img src="C:\Gitbook\Import\heart1ess_s_ctf\assets\MD5.png" alt="MD5"  />

* 衍生出对于已知hash值hash(salt)和salt长度但未知salt值的情况下可以知道hash(salt || padding || append)的值，其中padding为根据salt长度产生的填充，append为任意数据

* 由于md5在对原始数据进行512bit一组的分组后，采用初始向量（4*32bit）进行加密，每一轮的加密结果（128bit）分割为四组作为下一轮加密的初始向量，因而对于hash(salt)，我们通过填充padding从而让hash(salt)成为下一轮加密append的初始向量，从而完成对hash(salt || padding || append)的计算

* 可以采用工具hashpump

* 使用方式：![image-20210202114550739](C:\Gitbook\Import\heart1ess_s_ctf\assets\CSRF1.png)

* 其中Key Length指增加data之前的所有内容长度