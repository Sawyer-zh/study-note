# PHP编码规范及建议

1、PHP代码文件必须以 `<?php` 标签开始，并且不结尾

2、类/trait/Interface的命名必须遵循 `StudlyCaps `大写开头的驼峰命名规范

```php
class StudlyCaps {
}
trait StudlyCaps {
}
Interface StudlyCaps {
}
```

3、类中的常量所有字母都必须大写，单词间用下划线分隔

```php
define('FOO_BAR', 'something more');
const FOO_BAR = value;
```

4、方法(类/trait中，也就是说在类里面定义的函数我们把它们叫做方法)名称必须符合 `camelCase `式的小写开头驼峰命名规范

```php
class StudlyCaps {
    public function studlyCaps() {
        // coding...
    }
}
```

5、函数名称必须符合 `snake_case` 式的下划线式命名规范

```php
function snake_case() {
    // coding...
}
```

6、私有的(private)方法(类/trait中)名称必须符合 `_camelCase `式的前置下划线小写开头驼峰命名规范

```php
class StudlyCaps {
    private function _studlyCaps() {
        // coding...
    }
}
```

7、变量 必须符合 `camelCase `式的小写开头驼峰命名规范

```php
$someVariable = 'demo';
```











































