# ThinkPHP的一些技巧

1、一次插入多条记录的方法：

​	首先要够造出这样的一个多维数组：

​	array(

​	1 => array('column1' = > value1, 'column2' => value2, 'column' => value3),

​	2 => array('column1' = > value4, 'column2' => value5, 'column' => value6),

​	3 => array('column1' = > value7, 'column2' => value8, 'column' => value9),

​	........

​	);

然后再使用ThinkPHP自带的addAll方法即可，把上面这个多维数组传入这个addAll方法即可（如果插入成功，那么会返回1；失败，则返回0）