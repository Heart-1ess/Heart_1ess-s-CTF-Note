# Phar反序列化学习笔记

## 简介
PHAR (“Php ARchive”) 是PHP里类似于JAR的一种打包文件，在PHP 5.3 或更高版本中默认开启，这个特性使得 PHP也可以像 Java 一样方便地实现应用程序打包和组件化。一个应用程序可以打成一个 Phar 包，直接放到 PHP-FPM 中运行。

## Phar文件基本结构
### Stub
Stub的基本结构：`...<?php ...;__HALT_COMPILER()?>`...为用户自行填充内容。最终以`__HALT_COMPILER()?>`结尾，否则将无法识别为Phar文件。
### 