# ThinkPHP在第一重volist循环里定义的id要在第二重volist循环里面使用方法

例如：

<volist name="allTalks" id="talk" key='index'>

​	<volist name="talk.child" id="vo">

​	</volist>

</volist>

#### 其中的allTalks是一个二维数组，talk是一位数组，talk.child是talk里面的一维数组

#### name="talk.child"不能写成name="{$talk.child}"