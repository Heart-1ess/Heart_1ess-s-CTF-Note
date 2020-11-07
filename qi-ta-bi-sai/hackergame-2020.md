# Hackergame 2020

### 0x01 普通的身份认证器（JWT伪造）

登入网站发现只能使用Guest用户登录，需要是admin才能获取flag

抓包发现存在jwt，在burp suite的http历史中搜索jwt版本

```
#网址安全的JWT版本
eyJ[A-Za-z0-9_-]*\.[A-Za-z0-9._-]*

#所有JWT版本（误报的可能性更高）
eyJ[A-Za-z0-9_\/+-]*\.[A-Za-z0-9._\/+-]*
```

得到结果发现jwt版本存在漏洞![](/assets/hkg2.png)抓包token在 [https://jwt.io/ 上分析](https://jwt.io/上分析)

![](/assets/hkg1.png)

