# Laravel学习笔记



### 写每个功能的思路大致为：

### 一、理清有几张表和表之间的关系

### 二、创建migration文件、创建表的模型文件

### 三、理清路由有哪些

### 四、创建控制器和方法(同时看看控制器的方法里面是否要进行模型绑定，如果要就记得加上命名空间，例如:`use App\Topic;`)



### 1、创建项目的命令：`composer create-project laravel/laravel Laravel5 "5.4.*"`，其中`Laravel5`是项目名字(一个目录名)**

​	其中的`laravel/laravel`的意思：去`github`上面找`laravel/laravel`这个项目

​	其中的`Laravel5`是项目的名字

​	其中的`"5.4.*"`指的是用的laravel的分支(版本)是5.4.*

### **2、启动laravel项目的方法：**

​	方法一：使用php的内置服务器：`php -S localhost:8888 -t public`

​	方法二：使用Laravel提供的命令行工具(artisan)：`php artisan serve`

​			用方法二启动的默认端口是8000，我们可以指定端口，例如指定8888端口：`php artisan serve --port=8888`

### **3、路由文件是`app/Http/routes.php`文件(路由的参数最好写上命名空间)**

​	`routes.php文件:`

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/1.PNG)

​	`SiteController.php文件:`

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/2.PNG)

### **4、默认的视图文件是存放在：`resources/views/vendor`里面**

​	实际上，也可以在`views`文件夹的下面建立其他的文件夹，然后在view方法的里面指明路径

​	**注意：Laravel模板的文件默认是以`blade.php`结尾的**

### **5、创建控制器的命令：`php artisan make:controller SiteController`**(控制器、创建控制器、生成控制器)

​	然后我们可以在`/app/Http/Controllers`下面找到这个`SitesController.php`文件，这个文件里面都帮我们建立了一些方法。如果我们不想它帮我们建立一些方法的话，我们可以这样写：`php artisan make:controller SitesController --plain`

​	其中的`SitesController`可以换成其他的控制器名字

### **6、Laravel向视图传递变量(使用的是blade这个模板引擎)**

```php
<?php
//控制器里面的写法
namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;

class SitesController extends Controller {

    public function index() {
    	return view('welcome');
    }

    public function about() {
    	$name = 'jelly';
    	return view('sites.about')->with('name',$name);
    }
}

```

```php
<!DOCTYPE html>
<html>
<head>
	<title>Laravist.com</title>
</head>
<body>
  <!--模板里面的写法-->
	<h1>about me {{ $name }}</h1>
</body>
</html>
```

而对于不需要转义的值，我们使用两个!!来表示（特别是当我们使用`wangEditor`的时候，可以用这个方法）：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;

class SitesController extends Controller {

    public function index() {
    	return view('welcome');
    }

    public function about() {
    	$name = '<span style="color:red">jelly</span>';
    	return view('sites.about')->with('name',$name);
    }
}
```

```php
<!DOCTYPE html>
<html>
<head>
	<title>Laravist.com</title>
</head>
<body>
	<h1>about me {!! $name !!}</h1>

</body>
</html>
```

此时的结果：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/3.PNG)

此时blade**直接**把我们的值作为html输出

如果转义的话：

```php
<!DOCTYPE html>
<html>
<head>
	<title>Laravist.com</title>
</head>
<body>
	<h1>about me {{ $name }}</h1>

</body>
</html>
```

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/4.PNG)

此时，看到了变量原有的样子

如果要传递多个值的话，我们可以传递一个数组，作为view()方法的第二个参数

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;

class SitesController extends Controller {

    public function index() {
    	return view('welcome');
    }

    public function about() {
    	$data = array();
    	$data['first'] = 'Bool';
    	$data['last'] = 'Jelly';
    	return view('sites.about', $data);
    }
}
```

```php
<!DOCTYPE html>
<html>
<head>
	<title>Laravist.com</title>
</head>
<body>
	<h1>about me {{ $first }} {{ $last }} </h1>

</body>
</html>
```

还可以使用`compact`的方式传递变量：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;

class SitesController extends Controller {

    public function index() {
    	$posts = [
          [
            'title' => 'this is title1'
          ],
          [
            'title' => 'this is title2'
          ],
          [
            'title' => 'this is title3'
          ]
    	];
      return view('post/index', compact('posts'));
    }
}
```



### 7、Laravel环境配置**

在根目录下的`.env`文件里面

正式配置文件夹在根目录下的`config`文件夹里面

### **8、Migration**

在`database/migrations`文件夹下面

这是一个数据库版本管理

我们打开`2014_10_12_000000_create_users_table.php`文件可以看到：

```php
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('users');
    }
}
```

其中那个`create`方法是用来创建表的

里面的`$table->increments('id');`代表`id`字段是自增的

里面的`$table->string('name');`代表创建一个`string`类型的`name`字段

里面的` $table->rememberToken();`代表会创建一`remenber_token`字段

里面的` $table->timestamps();`会自动创建一个`created_at`和`update_at`两个字段

但是这样并没有创建`users`表，我们还需要执行以下一条命令：

```
php artisan migrate
```

### **9、把`laravel`添加到全局的方法**

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/5.PNG)

把这个画了圈的部分添加到家目录(home)下的`.bashrc`文件里面

### **10、使用`composer`如果太慢了的话，可以把把国外的镜像换成国内的镜像**

### **11、下载下来的包(插件)如果要使用的话，要先在`config/app.php`里面的`providers`数组里面引入**

### **12、Laravel文件夹：**

​	![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/6.PNG)

编辑最多的就是`app`这个文件

`public`存放的就是对外可见的资源，这里存放的`js`、`css`等等都可以被外界直接访问

`resources`存放的是模版文件，比如说`mvc`这一层的`v`都存放在这里

`routes`文件夹下存放的是路由文件，最重要的是：`web.php`这个文件，在`web.php`文件这里面存放是所有做网站要存放的路由

`storage`是和缓存有关(我们一般不用对这个文件夹进行改动，但是我们要确保启动laravel项目的人对这个文件夹有读写权限，否则会出错)

`tests`是测试用例

`vendor`是composer里面引用的所有的第三方包都可以在这里面找到

### **13、`config`下面的`database.php`里面的那个`env()`函数的意思：如果在`.env`文件里面没有定义这个常量，那么就使用这个`env()`函数后面那个默认的值**

​	我们修改配置都是在`.env`这个文件修改，而不会之间在`database.php`文件里面修改

### **14、路由参数**

​	例如：我们通过文章的`id`来定位显示不同的文章，这时候我们就要用到路由参数

```php
Route::get('articles/{id}', function() {
	return 'User'.$id;
});
```

其中这个`{}`叫做占位符，`id`占位在那里

### **15、路由分组**

```php
Route::group(['prefix' => 'admin'], function() {
	Route::get('users', function() {
		//匹配包含"/admin/users"的URL
    });
});
```

16、laravel的模板

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/7.PNG)

​	使用`yield`的时候，我们把公用的东西留下来，把不公用的东西删除，并且在删除的地方用上`yield`这个关键字

```php
@yield('content');
```

17、laravel建议表名是`名词 + 复数`的形式

18、laravel创建migration的方法(migration、创建migration)

`php artisan make:migration create_posts_table`

建议像`create_posts_table`这样命名migration

### (注意：posts一定要加上复数s)

如果想创建关系表的migration，可以：

```
php artisan make:migration create_post_topic_table
```

注意，在创建表的时候，表名还是要加上`s`(虽然migration文件的命名没有加上`s`)，例如：

```php
    public function up()
    {
        Schema::create('post_topics', function (Blueprint $table)
        {
            $table->increments('id');
            $table->integer('post_id')->default(0);
            $table->integer('topic_id')->default(0);
            $table->timestamps();
        });
    }
```

关系表创建模型的方法：

```
php artisan make:model PostTopic
```

### (注意：建立好模型之后，要马上指定哪些字段是可以被插入，哪些不能被插入！！！以免忘记)

19、laravel创建模型的方法(模型、创建模型)

```
php artisan make:model Post
```

### (其中Post是模型名，注意要大写)

然后就会在app目录下面创建一个名字叫做`Post.php`的文件名

里面的内容：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    //
}
```

注意，最好先使用属性`protected $fillable`指定一下哪些字段允许被插入数据

此时这个模型操作的数据表为：`posts`表

如果不是操作`posts`表的话而是想操作`posts2`表的话，就要指定它的`table`属性为`posts2`：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    protected $table = "posts2";
}
```

20、laravel默认的时区是在英国，我们可以把它设置为中国的时区

可以在`app/config/app.php`里面的`timezone`设置成`Asia/Shanghai`

21、tinker的使用

最经常使用的调试命令行的工具(注意：每次我们修改了代码之后需要重新启用`tinker`)

tinker可以模拟在模型里面写的代码，而不用直接把代码写在模型里面，起到了一个测试的作用

使用`php artisan tinker`来开启tinker，如下图：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/8.PNG)

这是一个shell，然后我们可以在里面直接写代码，我们输入如下语句：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/9.PNG)

然后就成功的在数据表里面插入了这条记录

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/10.PNG)

然后，如果我们要查找主键`id`为1的那条记录，可以使用：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/11.PNG)

**其中`find`只能用在主键上**

22、laravel核心思想-服务容器

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/12.PNG)

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/13.PNG)

在public目录里面的index.php文件中显示，laravel的容器是在`bootstrap/app.php`这个文件

其中容器是：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/14.PNG)

绑定的语句是：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/15.PNG)

获取容器是在`public/index.php`里面的语句：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/17.PNG)

使用(解析)容器的语句：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/18.PNG)

23、laravel核心思想-服务提供者

绑定一般是由服务提供者来做，一个服务提供者一旦注册了，它就提供一个服务

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/20.PNG)

laravel中提供的所有服务都是在`config/app.php`里面的`providers`写清楚了，还有一些是写死在框架里面的，我们可以在`Application.php`里面看到

24、laravel核心思想-门脸模式

可以静态调用容器里对应的类

25、数据填充(批量插入数据表的记录)

在路径`database/factories/ModelFactory.php`文件里

首先得在这个文件里面创建一个工厂：

例如，我想填充posts这张表，我可以这样写：

```php
$factory->define(App\Post::class, function(Faker\Generator $faker) {
	return [
		'title' => $faker->sentence(6),
		'content' => $faker->paragraph(10)
	];

});
```

写完后，打开`tinker`，输入如下命令：

```
factory(App\Post::class, 10)->make();
```



![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/21.PNG)

其中的`App\Post::class`指的是使用哪个工厂

其中的`10`指的是执行10次

其中的`make`指的是直接打印在终端上面

执行完后的结果：

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/22.PNG)



如果要插入到数据库中，要使用`create()`

```
factory(App\Post::class, 10)->create();
```

26、分页

首先在控制器里面使用`paginate`方法，并且指定每页显示的条数，例如：

```
$posts = Post::orderBy('created_at', 'desc')->paginate(6);
```

然后再在模板文件里面合适的位置加上：

```
{{ $posts->links() }}
```

27、在模板中截取一定长度的字符

可以使用`str_limit`这个函数：

```
{{ str_limit($post->content, 100, '...') }}
```

其中第一个参数是需要截取的内容

第二个参数是截取的长度

第三个参数是超出的内容以什么形式显示出来

28、模型绑定(绑定、绑定模型)

```php
Route::get('/posts/{post}', '\App\Http\Controllers\PostController@show');
```

这里的`{post}`说明模型绑定的是`Post`这个模型

#### (注意，页面上url传过去的是一个id，而在控制器里面的方法是通过模型来接收的，接收的时候回去posts表里面查id等于传过来的那个id，如果查不到，那么就会报一个没有查询结果的错误)

这个路由的含义是：对某一篇文章进行展示详情页

再比如：

```php
Route::post('/posts/{post}/comment', '\App\Http\Controllers\PostController@comment');
```

这个路由的含义是：对某一篇文章进行评论

所以，我们在控制器里面的`show`方法应该这样写：

```php

```

29、显示TokenMismatchException错误

(注意，凡是要提交表单的页面或者使用AJAX请求的时候，我们都要设置隐藏的token)

此时，我们应该在提交表单的时候，在里面写入

```html
<input type="hidden" name="_token" value="{{ csrf_token() }}" />
```

```html
<div class="create">
	<form action="/posts" method="POST">
		<input type="hidden" name="_token" value="{{ csrf_token() }}" />
		<div class="form-group">
			<label>title</label>
			<input type="text" name="title" class="form-control" placeholder="title">
		</div>
		<div class="form-group">
			<label>content</label>
			<textarea id="content" name="content" class="form-control" placeholder="content"></textarea>
		</div>
		<button type="submit" class="btn btn-default">submit</button>
	</form>
</div>
```



也可以用：

```
{{ csrf_field() }}
```

```html
<div class="create">
	<form action="/posts" method="POST">
		{{ csrf_field() }}
		<div class="form-group">
			<label>title</label>
			<input type="text" name="title" class="form-control" placeholder="title">
		</div>
		<div class="form-group">
			<label>content</label>
			<textarea id="content" name="content" class="form-control" placeholder="content"></textarea>
		</div>
		<button type="submit" class="btn btn-default">submit</button>
	</form>
</div>
```



### laravel建议在`html`的`head`标签里面加入：

```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

这样所有页面都有了

如果是使用AJAX请求的话，可以这样写`token`：

```

```



30、laravel插入记录到数据库的方法

```php
$post = new Post();
$post->title = request('title');
$post->content = request('contenet');
$post->save();
```

或者用更简单的方法：

```
Post::create(request(['title', 'content']));
```

注意，如果要使用`create`的方式插入数据，那么要在模型里面指明可以向数据表注入哪些字段！！例如：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

//it is posts table
class Post extends Model
{
	//protected $guarded

    protected $fillable = ['title', 'content'];
}
```

其中的`$guarded`是用来指定不可以注入的数据字段

其中的`$fillable`用来指定可以注入的数据字段

**如果写成`$guarded = [];`那么说明所有的数据字段都可以被插入**

31、所有的表单提交都要有的三个步骤：`验证`、`你想要执行的逻辑`、`渲染(比如说跳转到某个页面)`

32、storePublicly和asset函数的用法

```php
string|false storePublicly(string $path, array $options = array())
```

作用：

```
将上传的文件存储在具有公共可见性的文件系统磁盘上
```

```
string asset(string $path, bool $secure = null)
```

作用：

```
生成应用程序资源的URL
(根据目前请求的协定（HTTP 或 HTTPS）生成资源文件网址)
```

例子：

```php
public function imageUpload(Request $request)
{
    $path = $request->file('wangEditorH5File')->storePublicly(md5(time()));
  	return asset('store/'.$path);
}
```

33、验证表单的方法

可以使用laravel自带的方法，例如：

```php
public function register()
{
  $this->validate(request(), [
    'name' => 'required|min:3|unique:users,name',
    'email' => 'required|unique:users,email|email',
    'password' => 'required|min:5|max:10|confirmed'
  ]);
}
```

#### 键名是表单的`name`属性的值！！

`'name' => 'required|min:3|unique:users,name'`

里面的`required`说明这个字段是必须要填写的

里面的`min:3`说明这个字段至少要有3个字符

里面的`unique:users,name`说明这个传过来的在`users`表里面的`name`字段里面是唯一的

(注意，上面的三个条件只要有一个不满足，就会显示一个错误)



`'password' => 'required|min:5|max:10|confirmed'`

里面的`confirmed`说明这个传过来的name叫做`password`的内容要和传过来的name叫做`password_confirmation`的值要相同，否则会显示一个错误

(注意：验证条件之间不能加上空格！！否则可能会报错，即不能这样写`'required | min:3 | unique:users,name'`，因为在执行sql语句的时候，会在字段的旁边加上空格)

34、laravel对密码进行加密的方法

`bcrypt(密码)`

35、compact函数的用法

`compact()`函数创建一个由参数所带变量组成的数组。如果参数中存在数组，该数组中变量的值也会被获取

这个函数返回的数组是一个关联数组，键名为函数的参数，键值为产生中变量的值

例如：

```php
$first = 'Peter';
$lastname = 'Griffin';
$age = '38';

$result = compact('firstname', 'lastname', 'age');

print_r($result);
```

打印出的结果为：

```
Array
(
[firstname] => Peter
[lastname] => Griffin
[age] => 38
)
```

36、`request(['title', 'content'])`返回的是一个一维数组，键名分别为：`title`和`content`

37、Auth门脸类

```php
if(\Auth::attempt($user, $is_remember)) {
	return redirect('/posts');
}
```

如果要获取当前登录用户的所有信息，可以使用：

```
\Auth::user();
```

如果要获取当前登录用户的id，可以使用：

```
\Auth::id();
```

38、Redirect门脸类

```php
\Redirect::back()    //返回上一页
```

39、使用门脸类Auth的报错

```
Argument 1 passed to Illuminate\Auth\EloquentUserProvider::validateCredentials() must be an instance of Illuminate\Contracts\Auth\Authenticatable, instance of App\User given
```

40、如果要使用DB类来对数据库进行操作的话，要在控制器的上面写上`use DB`才行

41、查询

### 42、门脸类

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/23.PNG)

使用门脸类的话，要先配置模型(Model)，这时候，模型就不能是继承`App\Model`了，而应该继承`Illuminate\Foundation\Auth\User`

配置方法：

在User.php里面这样写即可：

```php
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;

//it is usrs table
class User extends Authenticatable
{
    //protected $guarded

    protected $fillable = ['name', 'email', 'password'];
}
```

```php
//登录的逻辑    
public function login()
    {
        $this->validate(request(), [
            'email' => 'required|email',
            'password' => 'required|min:5',
            'is_remember' => 'integer'
        ]);

        $user = request(['email', 'password']);
        $is_remember = boolval(request('is_remember'));


        if(\Auth::attempt($user, $is_remember)) {
            return redirect('/posts');
        }

        return \Redirect::back()->withErrors('failed');
    }
```

如果登录验证的用户信息不是在默认的users表的话，还需要配置`config/auth.php`下的`guards`(守卫)和`providers`，例如：

```php
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'admin' => [
            'driver' => 'session',
            'provider' => 'admins',
        ],

        'api' => [
            'driver' => 'token',
            'provider' => 'users',
        ],
    ],
    
    
        'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\User::class,
        ],

        'admins' => [
            'driver' => 'eloquent',
            'model' => App\AdminUser::class,
        ]

        // 'users' => [
        //     'driver' => 'database',
        //     'table' => 'users',
        // ],
    ],
```

然后，再在路由里面配置要登录验证后才能进去的路由，例如：

```php
	Route::group(['middleware' => 'auth:admin'], function() {
			//内容管理系统首页
			Route::get('/home', '\App\Admin\Controllers\HomeController@index');
	});
```

其中键名`middleware`的含义是中间介的意思，键值`auth:admin`中的`auth`说明我们使用的是Laravel的默认配置，然后再使用`guards`里面我们定义的`admin`

43、用户授权Policy

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/24.PNG)

权限问题(例如：谁能够对一篇文章进行修改和删除)

需要以上三个步骤

在`blade`模板里面如果要获取登录用户的用户名的话，可以使用`{{ \Auth::user()->name }}`来获取

```html
@can('update', $post)
	<!-- coding -->
@endcan
```

上面这段代码是写在模板里面的，表示如果当前用户有对文章进行更新的权限，那么就执行`coding`的部分

44、模型关联

使用模型关联的反向关联

下面这行代码是在Post模型里面写的，Post模型要关联User模型，写一个user方法，内容为：

```php
return $this->belongsTo('App\User', 'user_id', 'id');
```

其中第一个参数是关联的模型，第二个参数是外键，第三个参数是主键

实际上，如果遵循laravel的命名规范的话(外键是表名_id，主键命名为id)，那么可以简写为：

```php
return $this->belongsTo('App\User');
```

### (记得要使用return！！！)

关联完之后，我们就可以在post的模板里面这样写：

```html
{{ $post->user->name }}
```

从而得到$post(也就是这篇文章)的user(用户)的name(名字)

45、权限认证

首先要创建一个`policy`，例如我们要创建一个对文章的`policy`(权限)管理，那么可以使用命令行：

```
php artisan make:policy PostPolicy
```

执行完了这条命令之后，会在`app`的目录下创建一个`Policies`目录，里面有`PostPolicy.php`文件，我们定义的策略类如下：

```php
<?php

namespace App\Policies;

use App\User;
use App\Post;
use Illuminate\Auth\Access\HandlesAuthorization;

class PostPolicy
{
    use HandlesAuthorization;

    /**
     * Create a new policy instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    //修改权限
    public function update(User $user, Post $post)
    {
    	return $user->id === $post->user_id;
    }

    //删除权限
    public function delete(User $user, Post $post)
    {
    	return $user->id === $post->user_id;
    }
}
```

然后是注册策略类：

在`app/Providers/AuthServiceProvider.php`里面注册

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Gate;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        //把PostPolicy这个策略类注册到Post模型
        'App\Post' => 'App\Policies\PostPolicy'
    ];

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
```

然后在处理编辑的那个控制器里面加上一句：

```php
$this->authorize('update', $post);
```

此时，只有这篇文章的创建者才能对这篇文章进行更新

同理，在处理删除文章的那个控制器里面加上一句：

```php
$this->authorize('delete', $post);
```

46、报错Use of undefined constant layout - assumed 'layout' (View: /codeDir/laravelCode/jianhsu-app/resources/views/post/create.blade.php)

原因可能是在`@inlcude`的时候，没有在里面加上引号

### 47、laravel模型关联

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/25.PNG)

例如，我在Post模型里面这样写：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

//it is posts table
class Post extends Model
{
	//protected $guarded

    protected $fillable = ['title', 'content'];

    public function user()
    {
    	return $this->belongsTo('App\User');
    }

    public function comments()
    {
    	return $this->hasMany('App\comment');
    }
}
```

其中：

```php
public function comments()
{
	return $this->hasMany('App\comment');
}
```

上面这段代码把文章和评论关联起来之后，就可以获取一篇文章的所以评论

我们可以在模板里面这样写：

```html
{{ $post->user->name }}
```

(注意：在函数里面的user()是带有括号的，而我们在模板里使用user的时候就不要带括号)

如果我们想要按照评论的创建时间进行降序排列的话，可以使用`orderBy`来处理：

```php
public function comments()
{
	return $this->belongsTo('App\Post')->orderBy('created_at', 'desc');
}
```

因为一篇文章只能由一个用户编写，所以在命名的时候，` public function user()`中的`user`命名成单数

因为一篇文章可以有多个评论，所以在命名的时候，`public function comments()`中的`comments`命名成复数

**(注意：在控制器里面要获取到另外那张表(副表)的数据，要记得使用`get`方法，例如：`$settings = $user->settings()->get();`，则可以获取到settings表里面的数据，但是结果里面是没有主表的数据的！！其实也可以在模板里面来获取关联模型的数据，这样在控制器里面就可以少传递一个settings变量了，例如：`{{ $user->settings->description }}`)**

### 48、评论列表逻辑

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/26.PNG)

模型关联预加载的意思就是，我在模板里面使用关联模型渲染之前就已经对数据库进行了查询操作

```html
@foreach($post->comments as $comment)
<li class="list-group-item">
<h5>{{ $comment->created_at }} by {{ $comment->user->name }}</h5>
<div>{{ $comment->content }}</div>
</li>
@endforeach
```

此时，Post的关联模型是Comment，如果不使用模型关联预加载的话，那么，在执行`$post->comments as $comment`这条语句的时候，就会对数据库进行操作，这显然不符合`MVC`的思想(因为我们希望对数据库的操作是在模型里面进行的而不是在模板里面进行)

所以要在Post控制器里面执行`$post->load('comments');`：

```php
public function show(Post $post)
{
    $post->load('comments');
    return view('post/show', compact('post'));
}
```

注意，`$post->load('comments');`中的`comments`不能写成`comment`。因为，在Post模型里面的对Comment模型的关联书写的方式为：

```php
public function comments()
{
	return $this->hasMany('App\Comment');
}
```

`comments`方法是加了`s`的

### 49、文章列表页评论数(WithCount)

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/27.PNG)

例如，在获取Post模型的时候，使用`withCount()`方法(注意一定要写把模型关联起来了之后才能使用模型关联计数！！！)：

```php
public function index()
{
    $posts = Post::orderBy('created_at', 'desc')->withCount('comments')->paginate(6);
    return view('post/index', compact('posts'));
}
```

然后在模板里面写：

```html
{{ $post->comments_count }}
```

即可获得这篇文章的评论数

(注意`withCount()`只能在`where`语句中使用，例如可以用在`orderBy`，`find()`中)

可以`withCount`可以计算关联模型的数据，例如上面计算了一篇文章的评论评论数(comments)

### 注意：如果模型中一个关联的方法命名是有多个组成部分，那么，在模板里面就要使用下划线的方式来计数！！例如，方法定义为：

```php
    public function postTopics()
    {
        return $this->hasMany(\App\PostTopic::class, 'post_id', 'id');
    }
```

则模板里面要这样写才能获取数目：

```php
{{ $topic->post_topics_count }}
```

50、赞功能

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/28.PNG)

赞功能逻辑是放在模板里面判断的

一个用户对一篇文章进行赞就要在`zans`表里面插入一条数据

可以使用`firstOrCreate`方法来插入数据

### 51、取消赞

思路：首先要考虑`文章和赞这个模型是否有什么关联`，并且这个关联需要有一个参数--用户id

**一篇文章和一个用户只能产生一个赞的关联**，所以在Post模型中使用`hasOne`

```php
public function zan($user_id)
{
	return $this->hasOne(App\Zan::class)->where('user_id', $user_id);
}
```

其中`App\Zan::class`是字符串，等价于`'App\Zan'`

`where('user_id', $user_id)`的意思就是`zans`表的`user_id`等于`$user_id`的值

控制器里面取消赞的代码：

```php
public function unzan(Post $post)
{
    $post->zan(\Auth::id())->delete();
    return back();
}
```

其中`$post->zan(\Auth::id())->delete();`的含义：通过Post模型和Zan模型的关联，找到`zans`表中`user_id`为当前登录用户的`id`的那条记录，然后删除这条记录

(因为我们是在Post控制器里面操作，所以我们要让Post模型和Zan模型关联)

我们可以在模板里面这样写来判断是否存在对这篇文章的赞记录：

```html
@if($post->zan(\Auth::id())->exists())
<a href="/posts{{ $post->id }}/unzan" type="button" class="btn btn-default btn-lg">取消赞</a>
@else
<a href="/posts/{{ $post->id }}/zan" type="button" class="btn btn-primary btn-lg">赞</a>
@endif
```

`$post->zan(\Auth::id())->exists()`这句话的含义就是如果和这篇文章关联的`zans`表里面存在当前登录用户的赞记录

52、ORM中firstOrCreate的用法

```php
public function zan()
{
    $param = [
    'user_id' => \Auth::id(),
    'post_id' => $post->id
    ];

    Zan::firstOrCreate($param);
    reeturn back();
}
```

其中`Zan::firstOrCreate($param);`的含义：先查找`zans`数据表中是否有`'user_id' => \Auth::id()`和`    'post_id' => $post->id`这样一条数据，如果有的话，就查找出来；如果没有，我就创建

### (`firstOrCreate`特别适合点赞、关注等相关操作，这样就不会重复创建一条数据了)

53、如果没有指定提交信息的方式，那么默认是`GET`的方式，例如：

```html
@if($post->zan(\Auth::id())->exists())
<a href="/posts{{ $post->id }}/unzan" type="button" class="btn btn-default btn-lg">取消赞</a>
@else
<a href="/posts/{{ $post->id }}/zan" type="button" class="btn btn-primary btn-lg">赞</a>
@endif
```

这里点击链接的时候并没有指定提交数据的方式，但是，如果路由写成`POST`的话就会报错，即不能这样写：

```php
//文章点赞模块
Route::post('posts/{post}/zan', '\App\Http\Controllers\PostController@zan');
Route::post('posts/{post}/unzan', '\App\Http\Controllers\PostController@unzan');
```

而应该写成`GET`方式的路由：

```php
//文章点赞模块
Route::get('posts/{post}/zan', '\App\Http\Controllers\PostController@zan');
Route::get('posts/{post}/unzan', '\App\Http\Controllers\PostController@unzan');
```

54、MassAssignmentException的报错

##### 检查字段是否允许被插入！！

### 55、文章列表的喜欢数和评论数

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/29.PNG)







56、MySQL的模糊查询`like`对于需要中文分词(也就是按照中文的意思来分开，例如：我们都是中国人。如果搜索中国，那么会命中；如果搜索是中，则不会命中，因为是中没有含义。所以我们最好不使用mysql的like)的需求来说，它的效率和功能远远不够。此时可以使用专业的搜索引擎，例如：`Elasticsearch`，这是一个实时和分布式的搜索引擎。实时说明它的效率非常高，分布式说明它可扩展

​	`Elasticsearch`是基于**倒排索引**这个算法

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/30.PNG)

`Elasticsearch`里面的**索引**相当于mysql里面的database，比如我们可以给整个项目创建一个索引

每个索引下面有多个**类型**，这些类型相当于database里面的表

**文档**相当于表中的每一行数据，搜索需求里面，每一个文档就相当于一篇文章

**字段**，类似与title、content

模板，每个索引的每个字段都有哪些特殊的设置，并且在配置模板的时候，那指明这个模板要应用在那个索引上面，这样就把模板和索引对应上了

57、搜索模块实现基本步骤

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/31.PNG)

其中的`ik`是一个中文插件

其中的导入数据库已有的数据的意思：把数据库中所有的文章导入到`Elasticsearch`的索引中

![](http://oklbfi1yj.bkt.clouddn.com/Laravel%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/32.PNG)

#### 使用集成包的话，就不用单独再安装中文的插件了

58、关联模型的写法(关联模型、模型关联)

比如说Fan模型要关联User模型，可以这样写：

```php
//粉丝用户
public function fuser()
{
	return $this->hasOne(\App\User::class, 'id', 'fan_id');
}
```

其中`$this`指代当前模型`Fan`模型

`hasOne`这个函数的第一个参数是指定`Fan`模型要关联的模型(这里是User模型)，这是一个字符串，等价于`'App\User'`

`hasOne`指定了`Fan`模型和`User`模型的关系是一对一的

`hasOne`这个函数的第二个参数和第三个参数是关联的条件，第二个参数是要关联的模型的一个字段(也就是要关联的那个模型的一个外键)，第三个参数是当前模型的一个字段，它们的关系是相等的

**再举个例子：**

我在User模型里面写：

```php
//关注我的粉丝
public function fans()
{
	return $this->hasMany(\App\Fan::class, 'star_id', 'id');
}
```

这个关联的作用：返回当前用户的所有粉丝

判断是当前用户的粉丝的条件：Fan模型里面的`star_id`等于当前用户(模型)的`id`

然后，如果我要看当前用户是否被某个人(uid)给关注了，可以这样写：

```php
//当前用户是否被uid关注了
public function hasFun($uid)
{
	return $this->fans()->where('fan_id', $uid)->count();
}
```

其中`$this->fans()`的含义是返回当前用户的所有粉丝(因为我们上面已经关联了，并且定义了这个函数，所以这里可以直接用)

`where('fan_id', $uid)`是查询条件：`$uid`是否是在我的所有粉丝的`fan_id`的里面，如果在，就是当前用户的粉丝；否则不是

### 59、计算几天前的函数(diffForHumans())

可以在模板里面对时间使用：

```html
$post->created_at->diffForHumans()
```

这时候可以自动计算出那个时间是几天前

60、使用@include引入模板的时候可以传一个参数进去，例如：

```html
@include('user/badges/like' ['target_user' => $user])
```

其中`$user`是一个模型

### 61、视图合成器

我们一定是选择放在`app/Providers/AppServiceProvider.php`里面

并且可以放在里面`boot`方法里面

```php
    public function boot()
    {
        \View::composer('layout.sidebar', function($view){
            $topics = \App\Topic::all();
            $view->with('topics', $topics);
        });
    }
```

其中的`\View::compose`是门脸类的一个方法

其中的`'layout.sidebar'`是那个模板

其中的`$topics = \App\Topic::all();`是用来获取`Topic`表里面的所有`topic`

其中的`$view->with('topics', $topics);`是把取出来的`topic`注入到模板里面，类似于控制器向模板传变量

### (当很多页面都要使用同一个组件，并且这个组件的内容需要从数据库里面取出来的时候，就可以使用视图合成器)

62、如果要指定数据表的默认字符串长度(字符)的话，可以在`app/Providers/AppServiceProvider.php`的`boot`方法里面写：

```php
    public function boot()
    {
        //mb4string 767/4 = 191.xxx
        Schema::defaultStringLength(191);
    }
```

### 63、模型的scope

使用场景：向某个专题投稿文章

1、首先要投稿的文章必须是登录用户自己的文章

2、这篇文章必须是不属于这个专题的文章

在Laravel中，这种属于与不属于的限制在Laravel中有专门的方法来进行限制，也就是`scope`

`scope`是打印在模型上的，定义`scope`的方法：

1、首先在模型里面定义一个方法

​	方法名是Laravel定义的，也就是有规范的。例如：`scopeActive`这个方法名，第一部分的`scope`是必须的，第二部分的`Active`必须是首字母大写

​	方法的参数也是有规范的，例如：`scopeActive($query)`中的`$query`，方法中的第一个参数必须是一个查询的`build`语句，而第二个参数、第三个参数......这些后面的参数可以自定义

​	方法的返回值也是有规范的，即返回的也是一个查询的`build`语句，例如：`$query->where('active', 1`)`

一个完整的`scope`方法定义的例子：

```php
    //属于某个作者的文章
    public function scopeAuthorBy(Builder $query, $user_id)
    {
        return $query->where('user_id', $user_id);
    }

    public function postTopics()
    {
        return $this->hasMany(\App\PostTopic::class, 'post_id', 'id');
    }

    //还不在这个专题的文章
    public function scopeTopicNotBy(Builder $query, $topic_id)
    {
        return $query->doesntHave('postTopics', 'and', function($q) use($topic_id) {
            $q->where('topic_id', $topic_id);
        });
    }
```

`scope`使用的方法(在TopicController控制器里面)：

```php
    public function show(Topic $topic)
    {
    	//带文章数的专题
    	$topic = Topic::withCount('postTopics')->find($topic->id);

    	//专题的文章列表，按照创建时间倒叙排列，前10个
    	$posts = $topic->posts()->orderBy('created_at', 'desc')->take(10)->get();

        //属于我的文章，但是未投稿
        $myposts = \App\Post::authorBy(\Auth::id())->topicNotBy($topic->id)->get();

    	return view('topic/show', compact('topic', 'posts', 'myposts'));
    }
```

64、如果两张表的关系是多对多，那么要关联这两张表的模型的话，可以先建立一张关系表，然后使用`belongsToMany`的方法对关系表进行关联即可，例如：

```php
    public function posts()
    {
    	$this->belongsToMany(\App\Post::class, 'post_topics', 'post_id');
    }
```

65、报错Call to a member function getRelationExistenceCountQuery() on null

模型关联里面是否`return`回去了

66、Model是前后台公用的，所以不需要单独创建一个后台管理系统的Model管理目录

67、安装依赖

可以使用`composer require`命令来安装包，例如：

```
composer require 'almasaeed2010/adminlte=~2.0'
```

然后我们可以在`vendor`目录下面找到下载的这个包

68、如果在url里面的路由没有指定方法名，那么会自动去找控制器下面的`index`方法

**69、如果使用AJAX来传递数据进行表单验证的话，则Laravel内置的显示错误的方法就会失效**

**70、如果使用AJAX来进行传递登录验证的数据的话，那么，在后台是不能够重定向的，要用前端的js来跳转页面**

71、报错：Trying to get property of non-object (View: /codeDir/laravelCode/phpcom/resources/views/home/setting.blade.php)

可能是数据库里面没有要查的那条记录，从而导致在模板渲染的时候出现这个错误

所以，得在模板里面进行判断

判断变量是否存在的方法：

```php+HTML
{{ $name or 'Default' }}
```

在这个例子中，如果 `$name` 变量存在，它的值将会被显示出来。但是，如果这个变量不存在，便会显示 `Default`

### 72、同时使用`where`和	`get`方法获取的是二维数组，通过`find`获取的是一维数组

例如：

```php
$user = User::withCount('stars')->find($user->id);
```

它获取的是一维数组

而：

```php
$user = User::where('id', $user->id)->withCount('stars')->get();
```

它获取的是二维数组

虽然他们获取的记录结果是一样的，但是返回的数据结构是不同的

73、











































