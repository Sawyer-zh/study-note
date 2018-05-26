# Vue.js路由报错TypeError: Cannot read property '_c' of undefined

错误详情：

![](http://oklbfi1yj.bkt.clouddn.com/hexo/images/%E6%8D%95%E8%8E%B7.PNG)

像上面这个报错真的是是十分的蛋疼，根本看不懂，也定位不了，所以只能一行一行代码的去寻找可能出错的原因

先来看看我的代码（在main.js文件里面）：

```javascript
import Vue from 'vue'
import App from './App.vue'
import VRouter from 'vue-router'
import Apple from './components/apple.vue'
import Banana from './components/banana.vue'
Vue.use(VRouter);
let router = new VRouter({
  mode: 'history',
  routes: [
  	{
	  	path: '/apple',
	  	components: Apple
  	},
  	{
  		path: '/banana',
  		components: Banana
  	}
  ]
});

new Vue({
	el: '#app',
	router,
	render: h => h(App)
});
```

仔细排查错误后发现，routes里面的components多了一个s，正确的应该是：

```javascript
routes: [
  	{
	  	path: '/apple',
	  	component: Apple
  	},
  	{
  		path: '/banana',
  		component: Banana
  	}
  ]
```

所以一定要细心！！！

