# composer使用

## 1、使用中国镜像可以加快速度

```shell
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```

或者：

```shell
composer config repo.packagist composer https://packagist.phpcomposer.com
```

## 2、没有vendor

因为composer不能再root用户下执行，所以我们一般在普通用户下执行composer，但是普通用户可能没有写的权限，所以就无法创建文件夹，因此，需要在composer前面加上`sudo`：

```shell
sudo composer
```

## 3、./composer.json is not writable

和问题2一样，增加`sudo`：

```shell
sudo composer
```

## 4、composer.json可以自己创建

这个文件至少是这样的：

```Json
{
  
}
```

