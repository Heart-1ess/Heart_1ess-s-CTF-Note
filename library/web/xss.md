# XSS
* ### XSS跳转

代码示例：

```
<?php
echo "<script>window.location.href = $_GET['url']</script>";
?>
```

跳转至传入变量url的地点

