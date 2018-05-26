# 深入理解PHP原理之Opcodes

Opcode是一种PHP脚本编译后的中间语言

举个例子，比如你写下了如下的PHP代码：

```php
<?php
  echo "Hello World";
  $a = 1 + 1;
  echo $a;
```

PHP执行这段代码会经过如下4个步骤(确切的来说，应该是PHP的语言引擎Zend)



```
1.Scanning(Lexing) ,将PHP代码转换为语言片段(Tokens)
2.Parsing, 将Tokens转换成简单而有意义的表达式
3.Compilation, 将表达式编译成Opocdes
4.Execution, 顺次执行Opcodes，每次一条，从而实现PHP脚本的功能。
```

Lex就是一个词法分析的依据表。 Zend/zend_language_scanner.c会根据Zend/zend_language_scanner.l(Lex文件),来输入的 PHP代码进行词法分析，从而得到一个一个的“词”(源码中的字符串，字符，空格，都会原样返回)

Parsing阶段首先会丢弃Tokens Array中的多于的空格，然后将剩余的Tokens转换成一个一个的简单的表达式，如下：

```
1.echo a constant string
2.add two numbers together
3.store the result of the prior expression to a variable
4.echo a variable
```

Compilation阶段会把Tokens编译成一个个op_array, 每个op_arrayd包含如下5个部分：

```
1.Opcode数字的标识，指明了每个op_array的操作类型，比如add , echo
2.结果       存放Opcode结果
3.操作数1  给Opcode的操作数
4.操作数2
5.扩展值   1个整形用来区别被重载的操作符
```

比如，我们的PHP代码会被Parsing成:

```
* ZEND_ECHO     'Hello World'
* ZEND_ADD       ~0 1 1
* ZEND_ASSIGN  !0 ~0
* ZEND_ECHO     !0
```

#### ($a被优化成!0)









