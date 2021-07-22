# 从一道CTF题看PHP自动化语法分析

### ——通过AST抽象语法分析树对PHP进行批量序列化漏洞查找

## AST抽象语法树简介

AST (Abstract Syntax Tree) 是源代码的抽象语法结构树状表现形式，Webpack、ESLint、JSX、TypeScript 的编译和模块化规则之间的转化都是通过 AST 来实现对代码的检查、分析以及编译等操作。

而对于抽象语法树的在php代码审计的应用，不如通过一道CTF题目来进行分析。

强网杯2021初赛 pop_master

index.php源码如下：

```
<?php
include "class.php";
//class.php.txt
highlight_file(__FILE__);
$a = $_GET['pop'];
$b = $_GET['argv'];
$class = unserialize($a);
$class->VHUD0D($b);
```

随机生成的class.php，并且该文件有16万行，超过1万个类，我们需要从这一万个类中寻找出一条可以成功进行命令执行的pop链。

很显然在这种条件下人工去寻找并没有自动化分析来的迅速，那么应该如何进行自动化分析则是一个非常重要的问题。首先就是AST抽象语法分析树。

![AST图示](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/blogs/assets/ast1.png)

或者如下图所示：

![AST图示2](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/blogs/assets/ast2.png)

该种分析方法可以使得语法中各种元素被提取出来，从而方便我们对指定元素进行分析，如本题目中的类名以及类中包含的方法名。在php语言中采用`php-parser`构建抽象语法分析树。

而对于此题而言，采用AST抽象语法分析树并非全部，并且AST本身只负责对各个元素进行拆分，最重要的分析方面则是通过污点分析实现。

简而言之，污点分析采用自顶向下的原则，对于每棵语法树的枝叶和子结点进行分析，从可能被污染的原数据开始，递归地向下分析，若在遭遇有效消毒后则污点向下的子结点被消毒，不再被污点感染；而在未经历安全处理或经历无效安全处理后则污点向下的子结点继续被污染，当分析至存在敏感函数执行的语句时（如`eval`等）检查对应子结点的被污染程度，若被污染则证明存在漏洞。

![污点分析](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/blogs/assets/ast3.png)

因而对于此题而言，污点分析和ast抽象语法树的构建至关重要。对于以后的代码量较大的程序而言，该种自动化查找漏洞策略可大幅减少应用于漏洞挖掘方向上的人力与物力，使得漏洞挖掘更加自动化。
