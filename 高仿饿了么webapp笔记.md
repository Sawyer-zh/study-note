# 高仿饿了么webapp笔记

1、因为要开发移动端的webapp，它是不能被缩放的，所以要定义页面的视口为：

```

```

2、如果要在手机端看自己的页面，可以使用草料二维码

3、1px的写法最好是：

4、如果一个块（这个块包含一张图片）和另一个块（包含文字）在设置了display:inline-block之后，它们之间有一些空隙，那么是因为他们之间有一些空白字符，此时，可以在它们的父元素写font-size:0px;然后再在文字的那个块写上font-size: 14px;（这里的字体大小写你需要的就行了）

或者是让两个span标签不要换行也能达到消除间隙的效果

5、如果要设置span的宽高，那么要先把它写成display:inline-block;否则写宽高是无效的

6、如果有两个span，其中一个是display:inline-block;另一个是正常的，如果他们底部没有对齐，那么，可以设置display:inline-block的那个span顶部对齐，vertical-align:top

7、如果发现一个块是在某个块的右下角，考虑使用绝对定位而不是通过改变边距来实现，而且因为我们使用的是绝对定位，所以要指定它的宽高，这样才会被撑开

8、要在一句话或者一段话的超出部分使用省略号，可以使用下面的样式：

```css
white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
```

9、给图片加模糊效果可以使用；

```css
filter: blur(10px);/*或者其他像素*/
```

10、要给一个块加上另一个块作为背景的话，可以对那个背景图的块使用绝对定位并且要设置index为负数，并且要设置另一个块为relative，因为背景图是相对于这个relative的块来进行绝对定位的

11、全屏弹出层的整体应该是fix定位

```css
position: fixed;
z-index: 100;
top: 0;
left: 0;
width: 100%;
height: 100%;
overflow: auto;
background: rgba(7,17,27,0.8);
```

其中：

width: 100%;height: 100%;是相对于屏幕的

overflow: auto;不设成overflow: hidden;是因为如果弹出层的高度大于了屏幕，就不能向下滚动了

11、Css  Sticky  footers布局

12、控制块的显示和隐藏可以使用vue.js里面的v-show来轻松实现

13、可以把功能相似的地方抽象成一个组件

14、使用v-for来遍历的时候，首先可以肯定是它是遍历一个数组

15、flex有三个属性：第一个是等分，第二个是内容不足的一个缩放情况，第三个是占位宽度

​	设置某个块的样式为flex:1;它是会自适应的

16、给元素加样式的话，如果能够使用class尽量用class，尽量不要用标签去查找，因为用class的查找效率更高，性能更好

17、垂直居中最好的布局是使用display:table;

18、先写数据再布局

19、通过vue.js来获取dom的方法是，在那个dom元素上使用属性 ref，例如：

```html
<P ref="foods-wrapper">
  我是一个p
</P>
```

**注意，foods-wrapper不要写成驼峰命名，因为html对大小写不敏感**

那么，在vue.js里面，就可以使用下面的方法来获取：

```javascript
this.$refs.foodsWrapper
```

20、vue.js里面的nextTick干嘛用的

​	`this.$nextTick` 是一个异步函数，为了确保 DOM 已经渲染

在我们的实际工作中，列表的数据往往都是异步获取的，因此我们初始化 better-scroll 的时机需要在数据获取后

Vue 是数据驱动的， Vue 数据发生变化（`this.data = res.data`）到页面重新渲染是一个异步的过程，我们的初始化时机是要在 DOM 重新渲染后，所以这里用到了 `this.$nextTick`，当然替换成 `setTimeout(fn, 20)` 也是可以的。

当我们想要计算和dom相关的东西的时候，一定要把dom先渲染。虽然在vue里面，dom是数据的自然映射，我们改变了数据就改变了dom，但是dom发生真正变化其实是在nextTick这个回调函数之后。所以我们调用原生dom的时候，一定要调用nextTick这个接口，这样是更保险的

21、methods里面定义的方法可以在vue.js的其他地方使用，例如我有以下的一个方法：

```javascript
methods: {
  _initScroll() {
	......
  }
}
```

然后，我可以在vue.js里面的create()选项里面使用这个方法，使用的方法如下：

```
this._initScroll();
```

22、如果我们需要使用js来通过class属性来获取dom元素的话，可以这样做：

```html
<div class="food-list food-list-hook">
  ......
</div>
```

其中，我们想通过food-list-hook来获取它，而且，这里的food-list-hook是没有任何样式的，仅仅是为了被js获取。而这里所加的hook就是起到了一个提示作用：这个类名是没有样式作用的。（这是一个好的习惯）

23、为什么这里在 created 这个钩子函数里请求数据而不是放到 mounted 的钩子函数里？

因为 requestData 是发送一个网络请求，这是一个异步过程，当拿到响应数据的时候，Vue 的 DOM 早就已经渲染好了，但是**数据改变 —> DOM 重新渲染仍然是一个异步过程**，所以即使在我们拿到数据后，也要异步初始化 better-scroll。

24、计算属性会随时变化吗？会的

25、如果没有见到某个块，可以试着给他加上颜色先

26、基于黑色做透明是没有用的

27、要让某个字体图标水平居中，可以在包含字体的那个父元素写text-align:center;

28、`<router-view></router-view>`也是可以像其他组件一样可以传东西的，例如：

```vue
<router-view v-bind:seller="seller"></router-view>
```

29、注意，子组件从父组件接受传过来的值的时候，要写出对象的形式：

```javascript
props: {
			deliveryPrice: {
				type: Number,
				default: 0
			}
		}
```

不能直接写成：

```
props: {
			deliveryPrice
		}
```

而且，为了养成良好的编程习惯，我们要加上类型type和默认值default（其中defualt的意思是，如果父组件没有给子组件传递任何值，子组件也可以使用它的那个默认值，这里默认值是0）

30、计算属性算出来了之后要记得写return你需要的那个值

31、当一个块的样式是会随着某些条件变化的话，我们可以先找出它们的共性，然后再在这个基础上写样式，此时可以这样写：

```css
.logo{
/*这是共性*/
}
.logo.highlight{
  /*这是logo高亮时候的颜色*/
}
```

而且此时的highlight这个样式应该是通过vue.js的`v-bind:class=“{'highlight':totalCount>0}”`这种形式动态的加上去的

32、一般设置了子块的`display:inline-block;`之后，都要在父块里面设置`fontsize:0;`，并且还要在子块里面设置字体大小

33、当我们去给一个观测对象添加一个它不存在的字段的时候，直接去使用它是不行的。我们需要引入一个接口才行，例如

```javascript
<script type="text/javascript">
	import Vue from 'vue'   //全局引入

	export default{
		props: {
			food: {
				type: Object
			}
		},
		created() {

		},
		methods: {
			addCart() {
				if(!this.food.count){
					Vue.set(this.food, 'count', 1);//利用这个接口，给this.food增加一个属性count，并且设置它的值为1。设置完了之后，它的变化才能被观测到，才能通知dom发生变化
				}
				else{
					this.food.count++;
				}
			}
		}
	}
</script>
```

33、可以增加padding来增加可点击区域

34、写json格式的时候，最后一条不能加逗号

35、当要向子组件传递父组件的数据的时候，如果父组件的那个数据一开始是空的，那么，可以在父组件的子组件里面的标签上面写上`v-if="data.length>0"`延迟渲染dom，这样，子组件里面打印的时候就不会为空的了（实际上，当dom渲染完了之后，子组件也是可以获取到数据的）