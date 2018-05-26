---
title: 用PHP实现删除数据表中的一个节点（记录）以及它的子树
date: 2017-06-12 15:17:23
tags: 
- php 
- 算法 
- 递归
---

我们知道，在评论区中，经常会看见一个人评论之后，后面有人艾特前面的那个人，对他进行评论，然后又有人对上一个人进行评论
我们叫这种结构为**树形结构**，其中的一条评论我们可以称之为**节点**

![](http://oklbfi1yj.bkt.clouddn.com/hexo/images1.PNG)

这张数据表对应的树形结构是：

![](http://oklbfi1yj.bkt.clouddn.com/hexo/images2.PNG)

如果我要删除节点16，那么，这个树形图会变成

![](http://oklbfi1yj.bkt.clouddn.com/hexo/images3.PNG)

**不要在意树的箭头方向变了，我们关注的是树的逻辑结构的变化**

接下来用代码实现这个删除操作

```php
<?php

$con = mysqli_connect('localhost', 'root','');

if(!$con){

die('没有连到数据库'.msql_error());

}

mysqli_select_db($con, 'test');

    $sql = 'select * from treeNodes';
    $result = mysqli_query($con, $sql);
    if(!$result){
        die('查询失败！');
    }
    $row = array();//用来保存从数据库查询出来的记录
    $i = 0;
    while($da = mysqli_fetch_array($result, MYSQL_ASSOC)){
        $row[$i++] = $da;//一条一条的保存
    }
    
    delete_child_tree($con, $row, 16);//删除父id为16的子树
    
    
    function delete_child_tree($con, $data, $pid = 0){
        static $save_id = array();//用来保存需要删除的那个节点的id的数组
        $new_data = array();//用来保存删除节点后的新的数组
        foreach ($data as $key => $value) {
            if($value['pid'] == $pid){
                array_push($save_id, $value['id']);//保存需要删除的那个节点的id
                $sql = 'delete from treeNodes where id = '.$value['id'];
                mysqli_query($con, $sql);
            }
            else{
                $new_data[] = $value;//如果这个节点不用被删除的话，那个就保存他到新的数组里面
            }
    
        }
        var_dump($save_id, $new_data);
        if($id = array_shift($save_id)){
            echo '<span style="color:red;">'.$id.'</span>';
            delete_child_tree($con, $new_data, $id);//递归调用
        }
    }
    mysqli_close($con);

?>
```



执行这段代码之后，数据表变为：

![](http://oklbfi1yj.bkt.clouddn.com/hexo/images4.PNG)

至于id为16的那个节点，大家可以自己动手删掉它，也不难