# php 代码复用机制--traits

Traits 是一种为类似 PHP 的单继承语言而准备的代码复用机制。Trait 为了减少单继承语言的限制，使开发人员能够自由地在不同层次结构内独立的类中复用方法集。Traits 和类组合的语义是定义了一种方式来减少复杂性，避免传统多继承和混入类（Mixin）相关的典型问题

## 基础使用方法

Traits 的使用非常简单，只需要在类中使用 use 关键字即可

```php
trait A {
    public function test() {
        echo 'trait A::test()';
    }
}


class b {
    use A;
}
$b=new b();
$b->test();
```

## 优先级

简单来说 Trait 优先级大于父类方法，但是小于当前类方法

```php
trait A {
    public function test() {
        echo 'trait A::test()';
    }
    public function test1() {
        echo 'trait A::test1()';
    }    
}

class base{
    public function test(){
        echo 'base::test()';
    }
    public function test1(){
        echo 'base::test1()';
    }    
}
class b extends base{
    use A;
    public function test(){
        echo 'b::test()';
    }
}
$b=new b();
$b->test();//b::test()
$b->test1();//trait A::test1()
```

## Trait冲突问题

在使用多个 Trait 时，如果其中存在相同的方法名称，那么就会产生冲突。使用 insteadof 和 as 可以解决方法名称冲突问题

insteadof可以声明使用两个相同方法名称中的具体某个方法

```php
trait A {
    public function test() {
        echo 'trait A::test()';
    } 
}
trait B {
    public function test() {
        echo 'trait B::test()';
    } 
}
class c{
    use A,B{
        A::test insteadof B;//使用 insteadof 明确使用哪个方法
        B::test as testB;//使用 as 修改另外一个方法名称，必须在使用 insteadof 解决冲突后使用
    }
}
$c=new c();
$c->test();//trait A::test()
$c->testB();//trait B::test()
```

## 方法访问控制

使用 as 关键字我们可以对 trait 方法的访问权限进行修改

```php
trait A {
    public function test() {
        echo 'trait A::test()';
    } 
    private function test1(){
        echo 'trait A::test1()';
    }
}
class b{
    use A{
        test as protected;
        test1 as public test2;//更改权限时还可以修改名称
    }
}
$b=new b();
$b->test();//Fatal error: Call to protected method b::test()
$b->test2();//trait A::test1()
```

## Trait嵌套使用

```php
    trait A {
        public function test1() {
            echo 'test1';
        }
    }
     
    trait B {
        public function test2() {
            echo 'test2';
        }
    }
     
    trait C {
        use A,B;
    }
     
    class D {
        use C;
    }
     
    $d = new D();
    $d->test2();  //test2
```

## 变量、属性、方法定义

Trait可定义属性，但类中不能定义同样名称属性

```php
    trait A {
       public $test1;
    }
     
    class B {
        use A;
        public $test;
        public $test1;//Strict Standards: B and A define the same property ($test1) in the composition of B...
    }
```

Trait支持抽象方法、支持静态方法、不可以直接定义静态变量，但静态变量可被trait方法引用

```php
    trait A {
        public function test1() {
            static $a = 0;
            $a++;
            echo $a;
        }
     
        abstract public function test2(); //可定义抽象方法
    }
     
    class B {
        use A;
        public function test2() {
     
        }
    }
     
    $b = new B();
    $b->test1(); //1
    $b->test1(); //2
```

















































