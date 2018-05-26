# jquery学习笔记

1、使用`$.ajax()`从php返回中文数据的时候，出现乱码。可能的原因是没有指定：

```javascript
dataType: 'json',
```







