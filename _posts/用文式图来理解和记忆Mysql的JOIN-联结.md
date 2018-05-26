---
title: 用文式图来理解和记忆Mysql的JOIN(联结)
date: 2017-07-06 15:22:45
tags:
- 数据库
- Mysql
---

其中一些联结我们可以通过 `not null`这个查询条件来得到（例如，左外联结我们会得到一些null值在虚拟表中，所以我们可以通过 `not null` 来剔除那些 `null` 来得到左联结）

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8%E6%96%87%E5%BC%8F%E5%9B%BE%E6%9D%A5%E7%90%86%E8%A7%A3%E5%92%8C%E8%AE%B0%E5%BF%86Mysql%E7%9A%84JOIN%28%E8%81%94%E7%BB%93%29/1.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8%E6%96%87%E5%BC%8F%E5%9B%BE%E6%9D%A5%E7%90%86%E8%A7%A3%E5%92%8C%E8%AE%B0%E5%BF%86Mysql%E7%9A%84JOIN%28%E8%81%94%E7%BB%93%29/2.PNG)

<!-- more -->

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8%E6%96%87%E5%BC%8F%E5%9B%BE%E6%9D%A5%E7%90%86%E8%A7%A3%E5%92%8C%E8%AE%B0%E5%BF%86Mysql%E7%9A%84JOIN%28%E8%81%94%E7%BB%93%29/3.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8%E6%96%87%E5%BC%8F%E5%9B%BE%E6%9D%A5%E7%90%86%E8%A7%A3%E5%92%8C%E8%AE%B0%E5%BF%86Mysql%E7%9A%84JOIN%28%E8%81%94%E7%BB%93%29/4.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8%E6%96%87%E5%BC%8F%E5%9B%BE%E6%9D%A5%E7%90%86%E8%A7%A3%E5%92%8C%E8%AE%B0%E5%BF%86Mysql%E7%9A%84JOIN%28%E8%81%94%E7%BB%93%29/5.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8%E6%96%87%E5%BC%8F%E5%9B%BE%E6%9D%A5%E7%90%86%E8%A7%A3%E5%92%8C%E8%AE%B0%E5%BF%86Mysql%E7%9A%84JOIN%28%E8%81%94%E7%BB%93%29/6.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/%E7%94%A8%E6%96%87%E5%BC%8F%E5%9B%BE%E6%9D%A5%E7%90%86%E8%A7%A3%E5%92%8C%E8%AE%B0%E5%BF%86Mysql%E7%9A%84JOIN%28%E8%81%94%E7%BB%93%29/7.PNG)





