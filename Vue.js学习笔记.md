# Vue.js学习笔记

开发一般都是在src目录下进行



根目录下的index.html有一个默认js入口，也就是src目录下的main.js文件



src目录下的app.vue文件放置的是根组件



build和config目录是与配置有关



node_modules是node的依赖，main.js里面直接引入的vue都是从node_modules里面获取的

```javascript
import Vue from 'vue'
```



export作为输出后面必须跟着default，只有这样才能有以下写法：

```javascript
import Hello from './components/Hello'

export default{
  name: 'app',
  components: {
    Hello
  }
}
```

否则会报错



使用前端路由首先得安装前端路由，把vue-router安装到node_modules里面，同时package.json也得到了更新（在package.json里面的项目依赖dependencies可以看到）：

```
cnpm install vue-router --save
```

使用  --save  可以把它保存到配置文件（package.json）里

安装完成后，就可以在入口文件main.js引入我们的vue-router：

```javascript
import VRouter from 'vue-router'
```

然后就可以使用vue-router这个库（插件），使用下面的方法来注册使用我们的router：

```javascript
Vue.use(VRouter)
```

此时VRouter是一个全局的类



每个组件分为三个部分：

```vue
<template>
  ······
</template>
```

```vue
<script>
  ······
</script>
```

```vue
<style>
  ······
</style>
```



如果要在vue.js里面使用插件，例如：vue-router、vue-resource，要先在main.js里面注册，然后再来使用



1、new 一个Vue对象的时候你可以设置它的属性，其中最重要的包括三个，分别是data,methods,watch

2、其中data代表vue对象的数据，methods代表vue对象的方法，watch设置了对象监听的方法

3、Vue对象里的设置通过html指令进行关联

4、重要的指令包括

​	v-text   渲染数据

​	v-if   控制显示（条件渲染），元素会从文档流里面删除

​	v-else	紧接在v-if来配合使用

​	v-show	 控制显示（条件渲染），通过css的display:none来隐藏

​	v-on   绑定事件

​	v-for   循环渲染

​	v-bind	来绑定标签的属性名，可以简写为  :属性名

5、

```html
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <hello></hello>
    <h1 v-text="title"></h1>
    <input type="text" name="text" v-model="newItem" v-on:keyup.enter="addNew">
    <ul>
      <li v-for="item in items" v-bind:class="{finished: item.isFinished}" v-on:click="toggleFinish(item)">
        {{item.label}}
      </li>
    </ul>
  </div>
</template>

<script>
import Hello from './components/Hello'

export default {
  name: 'app',
  components: {
    Hello
  },
  data: function(){
    return{
      title: 'this is a todo list',
      items:[
        {
          label: 'coding',
          isFinished: true
        },
        {
          label: 'walking',
          isFinished: true
        }
      ]
    }
  },
  methods: {
    toggleFinish: function(item){
      item.isFinished = !item.isFinished;
      this.newItem = item.label
    },
    addNew: function(){
      this.items.push({ //这里的this是指上面定义的data对象，这个push函数可以把push中的参数（在这里是一个对象）放到items中
        label: this.newItem,//可以理解为v-model指令把newItem绑定到了上面的数据data中，也就是说data里面除了label和isFinished之外，还多了一个newItem
        isFinished: false
      })
      this.newItem = '';
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.finished{
  color: red;
}
</style>
```



6、

```javascript
//这是main.js文件
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'

Vue.config.productionTip = false

/* eslint-disable no-new */

//这个new Vue是根组件
// new Vue({
//   el: '#app',//组件要装载的位置，是index.html里的那个id为app的位置
//   template: '<App/>',//装载一个模板
//   components: { App }
// })

//全局注册组件的方法
/*
注册一个my-header的组件，
注册组件的时候不用使用new Vue的那种方法，直接使用{}即可
其中，注册组件的时候不用指明组件的挂载点，
也就是说不用写el这个属性

*/
Vue.component('my-header', {
	template: '<p>header</p>',
});	//

//测试，这是一个根组件
new Vue({
	el: '#app',
	template: '<h1>succuss, {{word}}</h1>',	//此时，index.html的那个app就会被替换为我这里的p标签，不过一般我们都不会直接在这里写template，而是直接在挂载点写模板
	data: {
		word: 'hello world2'
	}	//data是组件会用到的数据
});
```



```javascript
//这是main.js文件
import Vue from 'vue'

var myHeaderChild = {
	template: '<p>I am my header child</p>'
}

//把myHeader作为变量存起来，组件里面还有一个组件my-header-child
var myHeader = {
	template: '<p>this is my header<my-header-child></my-header-child></p>',
	components: {
		'my-header-child': myHeaderChild,
	}
}

/*
这是一个根组件
因为我们一般不会把所有的组件都注册为全局组件，
所以我们使用components这个属性来注册组件，
这样我们就可以在new Vue这个根组件的内部来使用这个组件了
*/
new Vue({
	el: '#app',
	data: {
		word: 'hello world2'
	},
	components: {//my-header是任意取的配置名
		'my-header': myHeader,	//让my-header这个组件关联到这个'my-header配置里面'
	}
});
```



7、

要避免组件的data是引用赋值，即：

```javascript
data: {
		f: 1,
		d: 2
}
```

应该以函数的形式赋值：

```javascript
data: function(){
  return {
    f: 1,
    d: 2
  }
}
```

如果我们通过引用赋值的方式的话，那么，如果一个相同的组件在多个地方使用，但是我只想改变一个地方的组件的data的值，此时，会把所有用到这个组件的地方的data值改变掉



8、

**全局API**

​	Vue的实例对象提供的全局方法，例如注册全局组件的Vue.component方法

**实例选项**（本质上是对象）

****	创建一个组件的时候，所传入的对象，例如Vue组件的**el、data**

**实例属性/方法**

​	都是以$符号开头的

**指令**

​	写在组件的模板（template）里面，通过模板与数据交互的方法

**内置组件**：

​	Vue提供的组件，例如：<component></component>、<keep-alive></keep-alive><router-view></router-view>、<transition></transition>组件

其中<keep-alive></keep-alive>一般配合<router-view></router-view>来使用，因为这样在组件切换的时候，就可以把组件缓存起来，以免下次切换的时候又去请求组件

<router-view></router-view>是路由视图



9、

```html
//在App.vue文件里面
/*
v-text和v-html用法的区别
使用表达式
*/
<template>
  <div>
    <p v-text="hello"></p>
    <p v-html="hello"></p>
    {{num + 1}}
    {{status ? 'success' : 'fail'}}
  </div>
</template>

<script>
export default{
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
    }
  }
}
</script>
```



10、

**v-for指令的用法**

使用这个指令来进行data里面的数组的渲染

```html
<template>
  <div>
      <ul>
        <li v-for="item in items">
          {{item.name}} - {{item.price}}
        </li>
      </ul>
      <ul>
        <li v-for="(item, index) in items" v-text="item.name + ' - ' + item.price">
        </li>
      </ul>
      <ul>
        <li v-for="(item, index) in items">
          {{item.name}} - {{item.price}} {{index}}
        </li>
      </ul>
  </div>
</template>
<script>
export default{
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
      items: [
        {
          name: 'apple',
          price: '34'
        },
        {
          name: 'banana',
          price: '50'
        }
      ]
    }
  }
}
</script>
```



11、

v-bind:class（或者:class）的用法

```html
<template>
  <div>
      <ul>
        <li v-for="(item, index) in items" v-bind:class="{odd:index % 2}">
          {{item.name}} - {{item.price}} {{index}}
        </li>
      </ul>
  </div>
</template>
<script>
export default{
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
      items: [
        {
          name: 'apple',
          price: '34'
        },
        {
          name: 'banana',
          price: '50'
        }
      ]
    }
  }
}
</script>
```

其中，index是从0开始计的，odd是一个类名，如果odd冒号后面的东西是0或者false，那么，不添加这个类，如果是非0或者true，那么，添加这个类



12、

使用v-for来进行data里面的对象的渲染

其中的index不一定是数字了，这一点和渲染数组的情况有些不同

value也可以替换成item或者其他的

```html
//在App.vue文件里面
<template>
  <div>
      <ul>
        <li v-for="(value, index) in objList">
          {{index}}: {{value}}
        </li>
      </ul>
  </div>
</template>
<script>
export default{
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
      objList: {
        name: 'apple',
        price: 34,
        color: 'red',
        weight: 14
      }
    }
  }
}
</script>
```



13、

我们一般都会把不同的组件放在不同的文件里面，而这些保存组件的文件后缀是.vue，一般都会放在src文件夹下面的components文件夹下面

例如，这是我的信建立的一个a组件，保存在a.vue文件里

```html
<template>
	<p>{{content}}</p>
</template>

<script type="text/javascript">
	export default{
		data: function(){
			return {
				content: 'I am a component'
			}
		}
	}
</script>
```

然后，我要在app组件（保存在app.vue文件里）里面去引入上面这个a组件：

```html
<template>
  <div>
      <ul>
        <li v-for="(value, index) in objList">
          {{index}}: {{value}}
        </li>
      </ul>
      <componentA></componentA>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA

export default{
  components:{componentA},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
      objList: {
        name: 'apple',
        price: 34,
        color: 'red',
        weight: 14
      }
    }
  }
}
</script>
```



14、

使用v-for指令来对组件列表循环渲染：

```html
<template>
  <div>
      <ul>
        <li v-for="(value, index) in objList">
          {{index}}: {{value}}
        </li>
      </ul>
      <componentA v-for="(value, index) in objList"></componentA>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA

export default{
  components:{componentA},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
      objList: {
        name: 'apple',
        price: 34,
        color: 'red',
        weight: 14
      }
    }
  }
}
</script>
```

因为objList有4项，所以，它会循环输出4次a组件



15、

列表数据同步更新方法

例如，使用push可以添加数据进入data里面的属性中

```html
<template>
  <div>
      <ul>
        <li v-for="item in items">
          {{item.name}} {{item.price}}
        </li>
      </ul>
    <button v-on:click="addItem">按钮</button>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA

export default{
  components:{componentA},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
      items: [
        {
          name: 'apple',
          price: 34
        },
        {
          name: 'banana',
          price: 50
        }
      ]
    }
  },
  methods: {//在方法里面可以调用组件里面的数据
    addItem(){
      this.items.push({
        name: 'orange',
        price: 39
      });
    }
  }
}
</script>
```

那么会把name: 'orange' 和 price: 39加入data里面的items数组列表中

直接对列表的某一项进行赋值，不会更新列表的数据，可以使用**set方法**（这是Vue的全局方法）去改变数组列表中的某一项的数据

------------------------------------------------------------------------------------------------------------------------------------------------

16、指令修改器

对v-on:keydown.enter=“func()”进行分析：

​	其中的v-on是指令，keydown是这个指令的参数，enter是指令修改器（意思是按下键盘上的enter键的时候触发函数），func()是触发的函数。

​	其中的enter是可以换成其他的指令修改器，例如数字13（v-on:keydown.13=“func()”）等等

​	如果我们把指令修改器给去掉（v-on:keydown=“func()”），此时，每当我们按下键盘上的任意键，都会去触发func()这个函数



17、自定义事件

​	我们在子组件a里面定义了一个自定义事件my-event，当my-event事件触发的时候，我们去执行onComaMyEvent函数

```html
//这好似a.vue文件
<template>
	<div>
		<p>{{content}}</p>
		<button v-on:click="emitMyEvent">emit</button>
	</div>
</template>

<script type="text/javascript">
	export default{
		data: function(){
			return {
				content: 'I am a component'
			}
		},
		methods:{
			emitMyEvent(){
				this.$emit('my-event', this.content);//这里定义了一个my-event事件，其中this.content是传递给app组件里的onComaMyEvent函数的参数
			}
		}
	}
</script>
```

```html
//这是app.vue文件
<template>
  <div>
      <ul>
        <li v-for="item in items">
          {{item.name}} {{item.price}}
        </li>
      </ul>
    <button v-on:click="addItem">按钮</button>
    <input type="text" name="text" v-on:keydown="onkeydown">
    <componentA v-on:my-event="onComaMyEvent"></componentA>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA

export default{
  components:{componentA},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
      items: [
        {
          name: 'apple',
          price: 34
        },
        {
          name: 'banana',
          price: 50
        }
      ]
    }
  },
  methods: {//在方法里面可以调用组件里面的数据
    addItem(){
      this.items.push({
        name: 'orange',
        price: 39
      });
    },
    onkeydown: function(){
      console.log('on key down');
    },
    onComaMyEvent(param){
      console.log('on Coma MyEvent' + param);
    }
  }
}
</script>
```



18、

**vue模板只能有一个根对象**

例如以下是错误的写法：

```html
<template>
    <el-button type="primary">{{test1}}</el-button>
    <button>点击我</button>
</template>
```

所以你想要出现正常的效果，你得用一个div来或是别的标签来包裹全部的元素：

```html
<template>
    <div
    	<el-button type="primary">{{test1}}</el-button>
    	<button>点击我</button>
    </div>
</template>
```



19、

v-model指令：表单数据模型双向绑定

​	v-model指令是用来绑定input标签的value属性的，我们知道，input标签的value属性就是input里面输入的值。然后，我们再让它和data里面的属性关联起来，这时候，我们就可以让input里面的值在其他地方使用，并且实现同步更新

```html
<template>
  <div>
      <ul>
        <li v-for="item in items">
          {{item.name}} {{item.price}}
        </li>
      </ul>
    <input type="text" name="text" v-model="myValue">
    <p>{{myValue}}</p>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA

export default{
  components:{componentA},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      hello: '<span style="color:red;">world</span>',
      num: 1,
      status: true,
      items: [
        {
          name: 'apple',
          price: 34
        },
        {
          name: 'banana',
          price: 50
        }
      ],
      myValue: '',
    }
  },
  methods: {//在方法里面可以调用组件里面的数据
    addItem(){
      this.items.push({
        name: 'orange',
        price: 39
      });
    },
    onkeydown: function(){
      console.log('on key down');
    },
    onComaMyEvent(param){
      console.log('on Coma MyEvent' + param);
    }
  }
}
</script>
```

**注意：我们必须先在data里面声明了myValue之后，才能写v-model="myValue"，否则会报错**

v-model指令也有它的修改器，例如：

```html
v-model.lazy="myValue"
```

使用了lazy这个修改器之后，只有当我们在input输入了值并且按下enter键或者input失去了焦点之后，才会在我上面的p标签里面输出myValue的值



20、属性监听

意思就是我们去监听一个vue.js的变量，每当这个变量被改动的时候，我们就去执行特定的操作

通过watch这个选项来实现的

当下面的例子中的myValue被改变的时候，会执行：

```javascript
myValue: function(val, old){
      this.newValue = val;
      this.oldValue = old;
    }
```

其中的val是传进去的myValue的新值，old是传进去的myValue的旧值

代码如下：

```html
<template>
  <div>
      <input type="text" name="text" v-model="myValue">
      <p>新的值：{{newValue}}</p>
      <p>旧的值：{{oldValue}}</p>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA

export default{
  components:{componentA},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      myValue: '',
      newValue: '',
      oldValue: ''
    }
  },
  methods: {//在方法里面可以调用组件里面的数据
    addItem(){
      this.items.push({
        name: 'orange',
        price: 39
      });
    },
  },
  watch:{
    myValue: function(val, old){
      this.newValue = val;
      this.oldValue = old;
    }
  }
}
</script>
```



21、

因为HTML默认对大小写不敏感，所以，我们在模板（template）中写组件的时候，建议使用中间带有横线的格式，例如：

我注册了两个组件：

```javascript
components: {
  ComA,
  comAHasSomeChildren
}
```

此时，我使用这两个组件的时候，建议用中线，例如：

```html
<template>
	<com-a></com-a>
	<com-a-has-some-children></com-a-has-some-children>
</template>
```

也就是说，每次遇到了一个大写，都要用 - 来进行分隔，并且都写成小写（同理，属性也是建议这种格式）。

在vue1.0如果不用这种格式是会报错的，但是vue2.0支持写成<ComA>、<comAHasSomeChildren>。因为它会转化为建议的那种格式。



22、

可以使用is标签**动态**的使用组件



23、

父组件要把他的一些值（例如number）传递给子组件，可以在子组件里面使用**props**选项来获取，并且在data里面不需要再声明它了。

在methods也可以通过this.number来使用它

假设，我在父组件里面使用了子组件，我可以在父组件里面的子组件标签里面写入需要传递的值

例如：

```html
<!--这是在父组件的app.vue文件里面-->
<template>
	<div>
		<!--向子组件传递这个number-->
		<com-a number=2></com-a>
	</div>
</template>
```



```html
<!--这是子组件所在的文件a.vue-->
<template>
	<div>
		<p>{{content}}</p>
		<p>我是来自·父组件的{{number}}</p>
	</div>
</template>

<script type="text/javascript">
	export default{
		props:['number'],
		data: function(){
			return {
				content: 'I am a component'
			}
		},
	}
</script>
```



注意一点，传递给子组件的那个值它要用中线的格式，不要用驼峰命名的格式，例如，

```html
<!--这是父组件所在的app.vue文件-->
<component-a num-to-do=3></component-a>
```



```html
<!--这是子组件所在的a.vue文件-->
props:['num-to-do']
```



```html
<!--这是a.vue文件，接下来使用传递过来的值-->
<template>
	<div>
		<p>我是来自·父组件的{{numToDo}}</p>
	</div>
</template>
```

注意到，在这里会有一个转换，因为num-to-do不符合js变量的命名，所以它会转换成numToDo

**{{}}里面放的是js变量**



还可以使用slot（插条）来传递，例如代码中的那个<p>我将被插入子组件中</p>

```html
<!--这是父组件所在的app.vue文件-->
<template>
  <div>
      <component-a num-to-do=3>
        <p>我将被插入子组件中</p>
      </component-a>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA

export default{
  components:{componentA},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      myValue: '',
      newValue: '',
      oldValue: ''
    }
  },
}
</script>
```

```html
<!--这是子组件所在的a.vue文件-->
<template>
	<div>
		<p>{{content}}</p>
		<p>我是来自·父组件的{{numToDo}}</p>
		<slot></slot>
	</div>
</template>

<script type="text/javascript">
	export default{
		props:['num-to-do'],
		data: function(){
			return {
				content: 'I am a component'
			}
		},
	}
</script>
```



如果在子组件里有多个slot，可以在父组件里面指定插入到那个slot里面



24、过渡的效果（使用css实现）

<transition></transition>配合v-show这样的指令来实现，例如：

```html
<!--这是app.vue文件-->
<template>
  <div>
      <button v-on:click="toggle">切换</button>
      <transition name="my-trans">
        <p v-show='isShow'>hello</p>
      </transition>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA

export default{
  components:{componentA},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      isShow: true,
    }
  },
  methods:{
    toggle(){
      this.isShow = !this.isShow;
    }
  }
}
</script>
<style type="text/css">
.my-trans-enter-active, .my-trans-leave-active{
  transition: .5s;
}
.my-trans-enter{
  transform:translateY(-500px);
  opacity: 0;
}
.my-trans-leave-active{
  transform: translateY(500px);
  opacity: 0;
}
</style>
```

分析：

​	过渡用四种状态：enter、enter-active、leave、leave-active

​	在进入时候，enter设置过渡开始态

​	元素本身的样式就是结束态

​	在离开时，元素本身就是离开时候的开始态

​	leave-active设置结束态

​	所以，一般都是只设置enter，leave-active一样就可以达到进入和退出的效果

​	



<transition name="my-trans">标签中的name值是可以任意取的，但是，对应的类名是要更改的，例如：

```html
<style type="text/css">
.my-trans-enter-active, .my-trans-leave-active{
  transition: .5s;
}
.my-trans-enter{
  transform:translateY(-500px);
  opacity: 0;
}
.my-trans-leave-active{
  transform: translateY(500px);
  opacity: 0;
}
</style>
```

因为<transition name="my-trans">标签中的name值是my-trans，所以，类名的定义要加上前缀my-trans，否则会但不到动画

不要漏了transition的时间，否则看不到切换的那个过程（动画）



25、动态的切换组件的显示

​	我们得把要切换的组件的挂载点放在<transition></transition>里面，例如：

```html
<template>
  <div>
      <button v-on:click="toggleCom">切换</button>
      <transition name="my-trans">
        <div v-bind:is="currentView"></div>
      </transition>
  </div>
</template>
<script>
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA
import componentB from './components/b.vue'

export default{
  components:{componentA, componentB},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      currentView: 'component-b',
      isShow: true,
    }
  },
  methods:{
    toggleCom(){
      if(this.currentView === 'component-a'){
        this.currentView = 'component-b';
      }
      else{
        this.currentView = 'component-a';
      }
    }
  }
}
</script>
<style type="text/css">
.my-trans-enter-active, .my-trans-leave-active{
  transition: .5s;
}
.my-trans-enter{
  transform:translateY(-500px);
  opacity: 0;
}
.my-trans-leave-active{
  transform: translateY(500px);
  opacity: 0;
}
</style>
```

如果我们只写成这样，我们会发现一个问题，比如所a组件还没有完全消失，b组件就跟着进来了(因此会有b组件下浮又上升的过程)。我们可以做如下修改：

```html
<transition name="my-trans" mode="out-in">
```

此时，我们很容易的想到，有没有in-out呢？

答案是有的，如果改成：

```
<transition name="my-trans" mode="in-out">
```

那么，当另一个组件完全显示出来了，另一个组件才会消失



26、多元素的过渡，如果两个元素的标签名是相同的，那么在vue里面是不会有动画效果的，例如：

```html
<template>
  <div>
      <button v-on:click="toggleCom">切换</button>
      <transition name="my-trans" mode="out-in">
        <p v-if="show">
          I am show
        </p>
        <p v-else>
          not in show
        </p>
      </transition>
  </div>
</template>
```

我们可以添加key来进行区分，例如：

```html
<template>
  <div>
      <button v-on:click="toggleCom">切换</button>
      <transition name="my-trans" mode="out-in">
        <p v-if="show" key="1">
          I am show
        </p>
        <p v-else key="2">
          not in show
        </p>
      </transition>
  </div>
</template>
```

注意，这里不是只能写key！！！这样也可以：

```html
<transition name="my-trans" mode="out-in">
  <p v-if="isShow" index="">I am show</p>
  <p v-else key="">not in show</p>
</transition>
```



27、过渡的效果（使用js实现，使用时间钩子）

（这种方法可以使用第三方库，例如jquery，建议把jquery的引入放在最外层的那个index.html文件里面）

使用js实现的时候，在<transition>标签里面是不需要name的，但是需要加上v-bind:css="false"

```html
<template>
  <div>
      <button v-on:click="isShow = !isShow">切换</button>
      <transition
      v-on:before-enter="beforeEnter"
      v-on:enter="enter"
      v-on:leave="leave"
      v-bind:css="false">
        <p class="animate-p" v-show="isShow">hello</p>
      </transition>
  </div>
</template>
<script>
import Vue from 'vue'
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA
import componentB from './components/b.vue'

export default{
  components:{componentA, componentB},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      currentView: 'component-b',
      isShow: true,
    }
  },
  methods:{
    beforeEnter: function(el){//其中的el指的是animate-p这个元素
      $(el).css({
        left: '-500px',
        opacity: 0
      });
    },
    enter: function(el, done){
      $(el).animate({
        left: 0,
        opacity: 1
      }, {
        duration: 1500,
        complete: done
      });
    },
    leave: function(el, done){//这里的done方法不能漏了，否则会出错
      $(el).animate({
        left: '500px',
        opacity: 0
      }, {
        duration: 1500,
        complete: done
      });
    }
  }
}
</script>
<style type="text/css">
html{
  height: 100%;
}
body{
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}
.animate-p{
  position: absolute;
  top: 0;
  left: 0;
}
</style>
```



28、自定义指令（分为局部的和全局的）

**使用选项directives**，例如：

```javascript
directives: {
  color: function(el, binding){
    el.style.color = binding.value
  }
}
```

这样就自定义了一个指令  v-color，它会是文字变成指令的值的颜色，具体的使用方法如下：

```html
//这是app.vue文件
<template>
  <div>
      <p v-color="'red'">hello world</p>
  </div>
</template>
<script>
import Vue from 'vue'
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA
import componentB from './components/b.vue'

export default{
  components:{componentA, componentB},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      currentView: 'component-b',
      isShow: true,
    }
  },
  directives: {
    color: function(el, binding){
      el.style.color = binding.value;
    }
  }
}
</script>
<style type="text/css">
html{
  height: 100%;
}
body{
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}
</style>
```



```html
<p v-color="'red'">hello world</p>
```

注意这里的  ‘red’  两边是需要加双引号的，如果没加，例如下面的样子会报错：

```html
<p v-color="red">hello world</p>
```

**以上是局部指令，也就是只能在app.vue文件中使用**

 如果需要写成全局指令，那么上面的内容就要写到main.js文件里



或许有人会问自定义指令有什么用，让字变成红色用v-bind内置的指令一样也可以做到啊。但是如果要实现光标自动聚焦的效果，Vue就用内置的指令实现不了了，但是可以使用自定义指令实现：

```html
<template>
  <div>
      <p v-color="'red'">hello world</p>
      <input type="text" name="text" v-focus>
  </div>
</template>
<script>
import Vue from 'vue'
import componentA from './components/a.vue' //引入这个a组件，组件名为from前面的名字componentA
import componentB from './components/b.vue'

export default{
  components:{componentA, componentB},//在引入之前，要先在当前的组件里先注册这个组件，这句话其实是ES6的缩写，他原来是{componentA:componentA}
  data: function(){
    return {
      currentView: 'component-b',
      isShow: true,
    }
  },
  directives: {
    color: function(el, binding){
      el.style.color = binding.value;
    },
    focus: {
      inserted(el, binding){
        el.focus();

      }
    }
  }
}
</script>
<style type="text/css">
html{
  height: 100%;
}
body{
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}
</style>
```



29、路由

```javascript
//main.js文件
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
	  	component: Apple
  	},
  	{
  		path: '/banana',
  		component: Banana
  	}
  ]
});

new Vue({
	el: '#app',
	router,
	render: h => h(App)
});
```

```vue
<!--app.vue文件-->
<template>
  <div>
    <router-view></router-view>
    <router-link v-bind:to="{path:'apple'}">apple</router-link>
    <router-link v-bind:to="{path:'banana'}">banana</router-link>
  </div>
</template>
<script>
import Vue from 'vue'

export default{
  name: 'app',
  components: {

  }
}
</script>
<style type="text/css">
html{
  height: 100%;
}
body{
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}
</style>
```

```vue
<router-view></router-view>是用来显示路由页面的
```

```vue
<router-link v-bind:to="{path:'apple'}">apple</router-link>的意思是点击了这个链接（实际上这就是一个a标签，我们可以查看源代码发现）之后，就会路由到apple那里，也就是在url会显示http://localhost:8080/apple
其实，如果不想动态绑定to属性的话，可以写成<router-link to="/apple">apple</router-link>注意要养成好习惯，不要漏了斜线，意思是根目录下的apple
```



上面的是最一般的路由方式，如果要传递参数的话，就要重新设置映射表：

```javascript
//main.js文件
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
	  	path: '/apple/:color',
	  	component: Apple
  	},
  	{
  		path: '/banana',
  		component: Banana
  	}
  ]
});

new Vue({
	el: '#app',
	router,
	render: h => h(App)
});
```

```vue
<!--apple.vue文件-->
<template>
	<div>
		<p>{{content}}</p>
		<button v-on:click="getParam">getParam</button>
	</div>
</template>

<script type="text/javascript">
	export default{
		data: function(){
			return {
				content: 'I am apple'
			}
		},
		methods: {
			getParam(){
				console.log(this.$route.params);
			}
		}
	}
</script>
```

此时，如果我在url栏中输入如下地址：

```
http://localhost:8080/apple/red
```

会在页面上出现：



![](http://oklbfi1yj.bkt.clouddn.com/6.PNG)



然后，当我点击getParam按钮的时候，就会在控制台中输出我在url中输入的参数：

![](http://oklbfi1yj.bkt.clouddn.com/7.PNG)

也可以在template里面直接通过{{$route.params.color}}来获取传进来的red参数



重定向的方法：

```javascript
//main.js文件
let router = new VRouter({
  mode: 'history',
  routes: [
  	{
  		path: '/',
  		redirect: '/apple'
  	},
  	{
	  	path: '/apple',
	  	component: Apple
  	},
  	{
  		path: '/banana',
  		component: Banana
  	}
  ]
});
```

其中的

```javascript
{
path: '/',
redirect: '/apple'
}
```

的意思是，当我访问根目录的时候，会路由到根目录下的apple



30、vuex（状态管理的一个插件）

```javascript
//main.js文件
import Vue from 'vue'
import VRouter from 'vue-router'
import App from './App.vue'
import Vuex from 'vuex'
import Apple from './components/apple.vue'
import Banana from './components/banana.vue'
Vue.use(VRouter)
Vue.use(Vuex)

let store = new Vuex.Store({
	state: {
		totalPrice: 0
	},
	mutations: {
		increment (state, price){
			state.totalPrice += price
		},
		decrement (state, price){
			state.totalPrice -= price
		}
	}
})

let router = new VRouter({
  mode: 'history',
  routes: [
  	{
  		path: '/',
  		redirect: '/apple'
  	},
  	{
	  	path: '/apple',
	  	component: Apple
  	},
  	{
  		path: '/banana',
  		component: Banana
  	}
  ]
});

new Vue({
	el: '#app',
	router,
	store,
	render: h => h(App)
});
```

上面的代码中，通过

```javascript
 new Vuex.Store
```

来实例化数据中心，state是数据中心需要存储的数据

mutations是动作，里面有两个动作，分别是increment和decrement，更改数据中心的状态只能用mutations里面的方法（动作）

所有的mutations函数里面都传入了两个参数

实例化好了store之后，我们把他放在全局使用（放置在全局实例化对象里面）

```javascript
new Vue({
	el: '#app',
	router,
	store,
	render: h => h(App)
});
```

放置在全局以后，就可以在每一个组件里面通过下面方法使用：

```javascript
this.$store.state.totalPrice
```



要使用数据中心的mutations动作要使用store的commit方法：

```vue
methods: {
			addOne (){
				this.$store.commit('increment', this.price)
			},
			minusOne (){
				this.$store.commit('decrement', this.price)
			}
		}
```



要使用数据中心的状态我们可以通过使用计算属性computed来返回：

```vue
<!--app.vue文件-->
export default{
  components: {
  	Apple,
  	Banana
  },
  computed: {
  	totalPrice (){
  		return this.$store.state.totalPrice
  	}
  }
}
```



总的代码为：

```javascript
//main.js
import Vue from 'vue'
import VRouter from 'vue-router'
import App from './App.vue'
import Vuex from 'vuex'
import Apple from './components/apple.vue'
import Banana from './components/banana.vue'
Vue.use(VRouter)
Vue.use(Vuex)

let store = new Vuex.Store({
	state: {
		totalPrice: 0
	},
	mutations: {
		increment (state, price){
			state.totalPrice += price
		},
		decrement (state, price){
			state.totalPrice -= price
		}
	}
})

let router = new VRouter({
  mode: 'history',
  routes: [
  	{
  		path: '/',
  		redirect: '/apple'
  	},
  	{
	  	path: '/apple',
	  	component: Apple
  	},
  	{
  		path: '/banana',
  		component: Banana
  	}
  ]
});

new Vue({
	el: '#app',
	router,
	store,
	render: h => h(App)
});
```

```vue
<!--apple.vue-->
<template>
	<div>
		<h1>{{content}}</h1>
		<button v-on:click="addOne">add one</button>
		<button v-on:click="minusOne">minus one</button>
	</div>
</template>

<script type="text/javascript">
	export default{
		data: function(){
			return {
				content: 'I am apple',
				price: 5
			}
		},
		methods: {
			addOne (){
				this.$store.commit('increment', this.price)
			},
			minusOne (){
				this.$store.commit('decrement', this.price)
			}
		}
	}
</script>
```

```vue
<!--banana.vue文件-->
<template>
	<div>
		<h1>{{content}}</h1>
		<button v-on:click="addOne">add one</button>
		<button v-on:click="minusOne">minus one</button>
	</div>
</template>

<script type="text/javascript">
	export default{
		data: function(){
			return {
				content: 'I am banana',
				price: 15
			}
		},
		methods: {
			addOne (){
				this.$store.commit('increment', this.price)
			},
			minusOne (){
				this.$store.commit('decrement', this.price)
			}
		}
	}
</script>
```

```vue
<!--app.vue文件-->
<template>
  <div>
    <img src="./assets/logo.png">
    {{totalPrice}}
    <apple></apple>
    <banana></banana>
  </div>
</template>

<script>
import Apple from './components/apple'
import Banana from './components/banana'

export default{
  components: {
  	Apple,
  	Banana
  },
  computed: {
  	totalPrice (){
  		return this.$store.state.totalPrice
  	}
  }
}
</script>
<style type="text/css">
html{
  height: 100%;
}
body{
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
}
.fade-enter-active, .fade-leave-active{
  transition: .5s;
}
.fade-enter{
  transform:translateY(-500px);
  opacity: 0;
}
.fade-leave-active{
  transform: translateY(500px);
  opacity: 0;
}
</style>
```



数据中心里面除了state和mutations之外，还有actions、getters和modules

actions是在mutations之前执行的，一般我们在actions里面调用mutations里面的动作，所以，actions起了一个中介的作用，此外，actions和mutations之间一个很重要的区别就是，actions可以异步请求(ajax)，而mutations不可以（mutations是同步的）



getters用来获取状态集里面的数据（state里面的）

