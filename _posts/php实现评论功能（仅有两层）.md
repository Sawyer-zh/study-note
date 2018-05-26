---
title: php实现评论功能（仅有两层）
date: 2017-06-12 16:21:53
tags: 
- php 
- 算法
---

**目前做的一个项目涉及到了说说功能**

以下是我的数据表：

![](http://oklbfi1yj.bkt.clouddn.com/hexo/images5.PNG)

其中可以看到，我只有两层评论，并没有那种很多层的复杂的树形结构

我现在想把这个杂乱的一系列评论分成一组一组的评论，其中，parent_id为0的作为第一层，parent_id与talk_id相同的作为第二层

开始我想的方法是递归来分出两层来，后来还是用比较暴力的方法来解决

以下是我的代码：

```php
function getSubTree($data = array(), $pid = 0){
	static $tmp = array();

	foreach($data as $key => $value){
		if($value['parent_id'] == $pid){
			$talk_id = $value['talk_id'];
			$value['child'] = getSubTree_1($data, $talk_id);
			$value['childCount'] = count($value['child']);
			$tmp[] = $value;
		}
	}
	return $tmp;
}

function getSubTree_1($data = array(), $pid){
	$tmp = array();
	foreach($data as $key => $value){
		if($value['parent_id'] == $pid){
			$tmp[] = $value;
		}
	}
	return $tmp;
}
```

思路是先在getSubTree函数里面找出parent_id为0的说说，然后再得到它的talk_id，然后再为这个说说（也就是这个parent_id为0的数组）构造一个一维数组child

接着再调用getSubTree_1函数来找出parent_id为$talk_id的说说，也就是第一层说说的评论

一直循环（找出parent_id为0的说说，然后再找这条说说的评论）

最终，我构造了一个新的数组（这个数组是三位数组，原来没有分类的时候是二维数组）