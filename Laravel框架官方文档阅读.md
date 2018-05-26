# Laravel框架官方文档阅读

### 1、public目录

包含了Laravel的HTTP入口文件 `index.php` 和前端资源文件（图片、JavaScript、CSS等）

### 2、storage 目录

包含编译后的 Blade 模板、基于文件的 session、文件缓存和其它框架生成的文件

`storage/app/public` 可以用来存储用户生成的文件，例如头像文件，这是一个公开的目录。你还需要在`public/storage` 目录下生成一个软连接指向这个目录，你可以使用 `php artisan storage:link` 来创建软链接

### 3、vendor 目录

目录包含所有 **Composer 依赖**

### 4、Mail 目录

`Mail` 目录默认不存在，但是可以通过执行 `make:mail` 命令生成，`Mail` 目录包含邮件发送类，邮件对象允许你在一个地方封装构建邮件所需的所有业务逻辑，然后**使用 `Mail::send` 方法发送邮件**

### 5、Providers 目录

`Providers` 目录包含应用的 **服务提供者** 。服务提供者在启动应用过程中绑定服务到容器、注册事件，以及执行其他任务，为即将到来的请求处理做准备

### 6、Laravel 的请求生命周期

#### 第一件事

一个 Laravel 应用的所有请求的入口都是 `public/index.php` 文件。 通过网页服务器 (Apache / Nginx) 所有请求都会导向这个文件。 `index.php` 文件没有太多的代码，只是加载框架其他部分的一个入口

`index.php` 文件载入 Composer 生成的自动加载器定义，并从 `bootstrap/app.php` 文件获取到 Laravel 应用实例。Laravel 的第一个动作就是创建一个自身应用实例 / 服务容器

#### HTTP / Console 内核

接下来，传入的请求会被发送给 **HTTP 内核**或者 console 内核，这根据进入应用的请求的类型而定。这两个内核服务是所有请求都经过的中枢。让我们现在只关注位于 `app/Http/Kernel.php` 的 HTTP 内核

HTTP 内核的标志性 `handle` 方法是相当简单的：接收一个 `Request` 并返回一个 `Response`。你可以把内核想成一个代表你应用的大黑盒子。给它喂 HTTP 请求然后它就会吐给你 HTTP 响应

##### 服务提供者

**在内核引导启动的过程中最重要的动作之一就是载入 服务提供者到你的应用**。所有的服务提供者都配置在`config/app.php` 文件中的 `providers` 数组中。 **首先，所有提供者的 `register` 方法会被调用，接下来，一旦所有提供者注册完成，`boot` 方法将会被调用**

**服务提供者负责引导启动框架的全部各种组件**，例如数据库、队列、验证器以及路由组件

服务提供者是 Laravel 应用的真正关键部分，应用实例被创建后，服务提供者就会被注册完成，并将请求传递给应用进行处理

##### 分发请求

一旦应用完成引导和所有服务提供者都注册完成，`Request` 将会移交给路由进行分发。路由将分发请求给一个路由或控制器，同时运行路由指定的中间件

### 7、服务容器

Laravel 服务容器是管理类依赖和运行依赖注入的有力工具。依赖注入是一个花俏的名词，它实质上是指：类的依赖通过构造器或在某些情况下通过「setter」方法进行「注入」

几乎所有服务容器的绑定都是在 服务提供者中进行的

> 但是，如果类没有依赖任何接口，那么就没有必要将类绑定到容器中了。容器绑定时，并不需要指定如何构建这些类，因为容器中会通过 PHP 的反射自动解析对象

### 8、服务提供者

服务提供者是所有 Laravel 应用程序引导启动的**中心所在**。包括您自己的**应用程序，以及所有的 Laravel 核心服务，都是通过服务提供者引导启动（一般是指注册事务）的**

#### 编写服务提供者

所有服务提供者都需要继承 `Illuminate\Support\ServiceProvider` 类

通过 Artisan 的 `make:provider` 命令行可以生成一个新的提供者：

```
php artisan make:provider RiakServiceProvider
```

##### 注册方法

在 `register` 方法中，**只能将事务绑定到 服务容器** 。不应该在 `register` 方法中尝试注册任何事件监听器，路由或者任何其他功能。否则，可能会意外的使用到尚未加载的服务提供者提供的服务

在服务提供者的方法中，都会提供一个有服务容器访问权限的 `$app` 属性，在你的任意一个服务提供者方法中，你总是可以通过访问 `$app` 属性使用服务容器

##### 引导方法

如果我们需要在我们的服务提供商中注册一个视图合成器，应该在 `boot` 方法中完成。**此方法在所有其他服务提供者均已注册之后调用**，这意味着您可以访问已由框架注册的所有服务

#### 注册提供者

所有服务提供者(包括我们自己编写的服务提供者)都在 `config/app.php` 配置文件中注册。此文件包含一个服务提供者类数组 **`providers`** 。默认情况下，它只会列出 Laravel 核心服务提供者类。这些服务提供者引导启动 Laravel 核心组件，例如邮件程序，队列，缓存和其他

要注册我们自己编写的服务提供者，只需将其添加到数组，例如：

```php
'providers' => [
    // Other Service Providers

    App\Providers\ComposerServiceProvider::class,
],
```

### 9、Facades 介绍

Facades（读音：/fəˈsäd/ ）为**应用程序的 服务容器 中可用的类**提供了一个「静态」接口

所有的 Laravel facades 都需要定义在命名空间 `Illuminate\Support\Facades` 下

#### 何时使用 Facades

使用 facades 最主要的风险就是会引起类作用范围的膨胀。因为 facades 使用起来非常简单而且不需要注入，我们会不经意的在单个类中大量使用。所以在使用 facades 的时候，要特别注意控制好类的大小，让类的作用范围保持短小

> 在开发与 Laravel 交互的第三方扩展包时，最好是在包中通过注入 Laravel contracts，而不是在包中通过 facades 来使用 Laravel 的类。因为扩展包不是在 Laravel 内部使用的，无法使用 Laravel's facade 的测试辅助函数

#### Facades 工作原理

在 Laravel 应用中，一个 facade 就是一个提供访问容器中对象的类。其中核心的部件就是 `Facade` 类。**不管是 Laravel 自带的 Facades ，还是用户自定义的 Facades ，都继承自 `Illuminate\Support\Facades\Facade` 类**

### 10、HTTP 路由功能

#### CSRF 保护

任何指向 `web` 中 `POST`, `PUT` 或 `DELETE` 路由的 HTML 表单请求都应该包含一个 CSRF 令牌，否则，这个请求将会被拒绝

#### 路由参数

##### 必选路由参数

路由的参数通常都会被放在 `{}` 内，并且参数名只能为字母，当运行路由时，参数会通过路由闭包来传递

> **注意：** 路由参数不能包含 `-` 字符。用下划线 (`_`) 替换 `-` 字符

##### 可选路由参数

声明路由参数时，如需指定该参数为可选，可以在参数后面加上 `?` 来实现，但是相应的变量必须有默认值：

```php
Route::get('user/{name?}', function ($name = null) {
    return $name;
});
```

```php
Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

#### 正则表达式约束

可以使用 `where` 方法来规范你的路由参数格式。`where` 方法接受参数名称和定义参数约束规则的正则表达式

#### 路由组

**路由组允许共享路由属性**，例如中间件和命名空间等，我们没有必要为每个路由单独设置共有属性，共有属性会以数组的形式放到 `Route::group` 方法的第一个参数中

##### 中间件

要给路由组中定义的所有路由分配中间件，可以在路由组中使用 `middleware` 键，中间件将会依照列表内指定的顺序运行

```php
Route::group(['middleware' => 'auth'], function () {
    Route::get('/', function ()    {
        // 使用 `Auth` 中间件
    });

    Route::get('user/profile', function () {
        // 使用 `Auth` 中间件
    });
});
```

##### 命名空间

为控制器组指定公共的 `PHP` 命名空间，使用 `namespace` 参数来指定组内所有控制器的公共命名空间：

```php
Route::group(['namespace' => 'Admin'], function () {
    // 在 "App\Http\Controllers\Admin" 命名空间下的控制器
});
```

> 请记住，默认 `RouteServiceProvider` 会在命名空间组中引入你的路由文件，让你不用指定完整的`App\Http\Controllers` 命名空间前缀就能注册控制器路由，因此，我们在定义的时候只需要指定命名空间`App\Http\Controllers` 以后的部分

##### 路由前缀

通过路由组数组属性中的 `prefix` 键可以给每个路由组中的路由加上指定的 URI 前缀，例如：

```php
Route::group(['prefix' => 'admin'], function () {
    Route::get('users', function ()    {
        // 匹配包含 "/admin/users" 的 URL
    });
});
```

##### 路由模型绑定

​	**隐式绑定**

​	Laravel 会自动解析定义在路由或控制器方法（方法包含和路由片段匹配的已声明类型变量）中的 Eloquent 模型

​	在这个例子中，由于类型声明了 Eloquent 模型 `App\User`，对应的变量名 `$user` 会匹配路由片段中的 `{user}`，这样，**Laravel 会自动注入与请求 URI 中传入的 ID 对应的用户模型实例(注意，默认绑定主键id)**

​	**如果数据库中找不到对应的模型实例，将会自动生成产生一个 404 HTTP 响应**

##### 表单方法伪造

**注意**：HTML 表单没有支持 `PUT`、`PATCH` 或 `DELETE` 动作

所以，当我们定义了`put`、`patch` 或`delete`的路由时候，需要在表单中增加隐藏的 `_method` 字段。 `_method` 字段的值将被作为 HTTP 的请求方法使用：

```html
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

### 11、路由中间件

Laravel 中间件提供了一种方便的机制来过滤进入应用的 HTTP 请求，例如，Laravel 包含验证用户身份权限的中间件。如果用户没有通过身份验证，中间件会重定向到登录页，引导用户登录

除了身份验证以外，中间件还可以被用来执行各式各样的任务

Laravel 已经内置了一些中间件，包括身份验证、CSRF 保护等。**所有的中间件都放在 `app/Http/Middleware` 目录内**

**我们可以将中间件想象为一系列的「层」，HTTP 请求必须经过它们才会触发我们的应用程序**。每一层都可以检测接收的请求，甚至可以完全拒绝请求访问您的应用

#### 创建中间件

使用命令：

```
php artisan make:middleware CheckAge
```

之后将会在 `app/Http/Middleware` 目录内新建一个 `CheckAge` 类

我们可以在这个类里面限制访问该页面的年龄

#### 注册中间件

##### 全局中间件

如果你希望访问你应用的每个 HTTP 请求都经过某个中间件，只需将该中间件类列入 `app/Http/Kernel.php` 类里的`$middleware` 属性

##### 为路由指定中间件

为特殊的路由指定中间件，首先应该在 `app/Http/Kernel.php` 文件内为该中间件指定一个 `键值`（在`$routeMiddleware` 属性）

##### 中间件

有时可能想要将多个中间件分组到同一个`键值`下，从而使它们更方便地分配给路由，你可以使用 HTTP kernel 的`$middlewareGroups` 属性来实现

例如，Laravel 带有开箱即用的 `web` 和 `api` 中间件，包含了可能应用到 Web UI 和 API 路由的通用中间件：

```php
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        'throttle:60,1',
        'auth:api',
    ],
];
```

中间件组可以像单个中间件一样使用相同的语法指定给路由和控制器

```php
Route::group(['middleware' => ['web']], function () {
    //
});
```

路由组仅仅是为了使一次将多个中间件指定给某个路由的实现变得更加方便

### 12、伪造跨站请求保护 CSRF

跨站请求伪造是一种恶意的攻击，它凭借已通过身份验证的用户来运行未经过授权的命令

Laravel 为每个活跃用户的 Session 自动生成一个 CSRF 令牌。该令牌用来核实应用接收到的请求是通过身份验证的用户**出于本意发送的**

**任何情况下在你的应用程序中定义 HTML 表单时都应该包含 CSRF 令牌隐藏域，这样 CSRF 保护中间件才可以验证请求**

#### X-CSRF-TOKEN

除了检查 POST 参数中的 CSRF token 外，`VerifyCsrfToken` 中间件还会检查 `X-CSRF-TOKEN` 请求头。你可以将令牌保存在 HTML `meta` 标签中：

```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

### 13、HTTP 控制器

> 控制器并不是 **一定** 要继承基础类（即Controller控制器）。只是，你将无法使用一些便捷的功能，比如 `middleware`，`validate`和 `dispatch` 方法

我们在定义控制器路由时，不必指定完整的控制器命名空间。`RouteServiceProvider` 会在一个包含命名空间的路由组中加载路由文件，因此我们只需要指定类名中 `App\Http\Controllers` 命名空间之后的部分就可以了

#### 控制器中间件

虽然中间件可以在路由文件中指定给控制器路由，例如：

```php
Route::get('/profile', 'UserController@show')->middleware('auth');
```

但是，在控制器的构造方法中指定中间件会更为便捷(甚至可以约束中间件只对控制器类中的某个特定方法生效)，例如：

```php
class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log')->only('index');

        $this->middleware('subscribed')->except('store');
    }
}
```

> 你可能将中间件指定到控制器的部分操作上，然而，这会使你的控制器过于臃肿。换个角度，考虑将控制器分成多个更小的控制器

#### 依赖注入与控制器

##### 方法注入

一个常见的用法就是将`Illuminate\Http\Request` 实例注入控制器方法中，例如：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $name = $request->name;

        //
    }
}
```

如果控制器方法需要从路由参数中获取输入内容，只需在其他依赖后列出路由参数即可，例如：

```php
Route::put('user/{id}', 'UserController@update');
```

通过以下方式定义控制器方法，可以让你在使用 `Illuminate\Http\Request` 类型约束的同时仍然可以获取参数`id`：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    public function update(Request $request, $id)
    {
        //
    }
}
```

#### 路由缓存

> 基于闭包的路由并不能被缓存。如果要使用路由缓存，你必须将所有的闭包路由转换成控制器类

使用路由缓存将极大地减少注册全部应用路由的时间。某些情况下，路由注册甚至可以**快一百倍**

生成路由缓存的命令：

```
php artisan route:cache
```

**如果添加了新的路由，你需要刷新路由缓存**

因此，你应该只在项目部署时才运行 `route:cache` 命令

清除路由缓存命令：

```php
php artisan route:clear
```

### 14、HTTP 请求 Request

#### 获取请求

如果要通过依赖注入的方式来获取当前 HTTP 请求的实例的话，应该在控制器方法中使用 `Illuminate\Http\Request` 类型提示

#### 请求路径 & 方法

##### 获取请求路径

`path` 方法返回请求路径信息：

```php
$uri = $request->path();
```

##### 获取请求方法

`method` 方法将返回 HTTP 的请求方式:

```php
$method = $request->method();
```

 `isMethod` 方法去验证 HTTP 的请求方式与指定规则是否相配：

```php
if ($request->isMethod('post')) {
    //
}
```

#### 获取输入数据

##### 获取所有输入数据

```php
$input = $request->all();
```

##### 获取指定输入值

```php
$name = $request->input('name');
```

如果传输表单数据中包含「数组」形式的数据，那么可以使用「点」语法来获取数组：

```php
$name = $request->input('products.0.name');
$names = $request->input('products.*.name');
```

##### 获取 JSON 输入信息

可以通过 「点」语法来读取 JSON 数组：

```php
$name = $request->input('user.name');
```

##### 确定是否有输入值

当数据存在　**并且**　字符串不为空时，`has` 方法就会返回`true`：

```php
if ($request->has('name')) {
    //
}
```

#### Cookies

##### 从请求中获取 Cookie 值

```php
$value = $request->cookie('name');
```

**Laravel 框架创建的每个 cookie 都会被加密**并且加上认证标识，这代表着用户擅自更改的 cookie 都会失效

#### 文件资源

##### 获取上传文件

```php
$file = $request->file('photo');
```

确认上传的文件是否存在：

```php
if ($request->hasFile('photo')) {
    //
}
```

#### 储存上传文件

`store` 方法允许存储文件到相对于文件系统根目录配置的路径。这个路径不能包含文件名，名称将使用 MD5 散列文件内容自动生成：

```php
$path = $request->photo->store('images');
```

`store` 方法还接受一个可选的第二个参数，用于文件存储到磁盘的名称。这个方法会返回文件相对于磁盘根目录的路径：

```php
$path = $request->photo->store('images', 's3');
```

### 15、Laravel 的请求返回 Response

#### 创建响应

框架也会自动地将数组转为 JSON 响应：

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

> Eloquent 集合也可以从路由和控制器中直接返回，它们会自动转为 JSON 响应

#### 重定向

```php
return redirect('home/dashboard');
```

重定向至上级一页面：

```php
 return back()->withInput();
```

**(此功能利用了 Session，请确保调用 `back` 函数的路由是使用 `web` 中间件组或应用了所有的 Session 中间件)**

##### 重定向至命名路由

如果路由有参数，它们可以作为 `route` 方法的第二个参数来传递：

```php
// 对于该 URI 的路由: profile/{id}
return redirect()->route('profile', ['id' => 1]);
```

##### 重定向至控制器行为

```php
return redirect()->action('HomeController@index');
```

如果控制器路由包含参数则需要把他们作为 `action` 函数的第二个参数传递：

```php
return redirect()->action(
    'UserController@profile', ['id' => 1]
);
```

#### 视图响应

如果响应内容不但需要控制响应状态码和响应头信息而且还需要返回一个 视图，这时你应该使用 `view` 方法

```php
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```

(如果不需要自定义 HTTP 状态码和响应头信息，则可使用全局的 `view` 辅助函数)

#### JSON 响应

自动将 `Content-Type` 响应头信息设置为 `application/json`，并使用 PHP 的 `json_encode` 函数将数组转换为 JSON 字符串：

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA'
]);
```

#### 文件下载

`download` 方法可以用于生成强制让用户的浏览器下载指定路径文件的响应。`download` 方法接受文件名称作为方法的第二个参数，此名称为用户下载文件时看见的文件名称。最后，你可以传递一个包含 HTTP 头信息的数组作为第三个参数传入该方法：

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

#### 文件响应

用来显示一个文件：

```php
return response()->file($pathToFile);
```

### 16、Laravel 的视图功能

#### 创建视图

通过全局函数 `view` 来使用视图：

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

第一个参数是 **`resources/views` 目录中**视图文件的文件名，第二个参数是一个数组

##### 判断视图文件是否存在

```php
use Illuminate\Support\Facades\View;

if (View::exists('emails.customer')) {
    //
}
```

#### 传递数据到视图

使用**数组**将数据传递到视图文件

##### 把数据共享给所有视图

使用 `View` Facade 的 `share` 方法。通常需要将所有 `share` 方法的调用代码放到 服务提供者 的 `boot` 方法中，此时你可以选择使用 `AppServiceProvider`或创建独立的 服务提供者

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        View::share('key', 'value');
    }

    public function register()
    {
      //coding
    }
}
```

#### 视图合成器

视图合成器是在视图渲染时调用的一些回调或者类方法。如果你需要在某些视图渲染时绑定一些**数据**上去，那么视图合成器就是你的的不二之选，使用视图合成器可以将代码组织到一个地方

**注册视图合成器**(在一个 服务提供者 中注册一些视图合成器，使用的是回调的形式)：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    public function boot()
    {
         View::composer('profile', function($view) 
         { 
             $view->with('count', User::count()); 
         }); 
    }

    public function register()
    {
        //
    }
}
```

基于类的视图合成器：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function boot()
    {
        View::composer(
            'profile', 'App\Http\ViewComposers\ProfileComposer'
        );
    }

    public function register()
    {
        //
    }
}
```

对应的控制器：

```php
<?php

namespace App\Http\ViewComposers;

use Illuminate\View\View;
use App\Repositories\UserRepository;

class ProfileComposer
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

每当视图渲染时，该合成器的 `compose` 方法都会被调用(注意：**默认是调用`compose`方法**)，使用 `with` 方法绑定数据到目标视图(也就是此例子中的`profile.blade.php`视图)

##### 将视图合成器附加到多个视图

将目标视图文件数组作为第一个参数传入 `composer` 方法：

```php
View::composer(
    ['profile', 'dashboard'],
    'App\Http\ViewComposers\MyViewComposer'
);
```

`composer` 方法同时也接受通配符 `*` ，可以让你将一个视图合成器一次性绑定到所有的视图

### 17、HTTP 会话机制

由于 HTTP 是无状态的，Session 提供了一种在多个请求之间存储有关用户信息的方法

Laravel 附带了几个不错且可开箱即用的驱动：

- `file` - 将 Session 保存在 `storage/framework/sessions`。
- `cookie` - Session 保存在安全加密的 Cookie 中。
- `database` - Session 保存在关系型数据库。
- `memcached` / `redis` - 将 Sessions 保存在其中一个快速且基于缓存的存储系统中。
- `array` - 将 Sessions 保存在简单的 PHP 数组中，并只存在于本次请求.

#### 从 Session 中取出并删除数据

```php
$value = $request->session()->pull('key', 'default');
```

(其中，key代表键名)

#### 闪存数据到 Session

存入一条缓存的数据，让它只在下一次的请求内有效，则可以使用 `flash` 方法。使用这个方法保存 session，只能将数据保留到下个 HTTP 请求，然后就会被自动删除：

```php
$request->session()->flash('status', 'Task was successful!');
```

#### 删除 Session 数据

从 Session 内删除一条数据：

```php
$request->session()->forget('key');
```

删除 Session 内所有数据：

```php
$request->session()->flush();
```

### 18、表单验证机制详解

`validate` 方法会接收 HTTP 传入的请求以及验证的规则。如果验证通过，你的代码就可以正常的运行。若验证失败，则会抛出异常错误消息并自动将其返回给用户。在一般的 HTTP 请求下，都会生成一个重定向响应，而对于 AJAX 请求则会发送 JSON 响应

#### 嵌套属性的注解

如果 HTTP 请求包含一个 「嵌套的」 参数，你可以在验证规则中通过 「点」 语法来指定这些参数：

```php
$this->validate($request, [
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

#### 显示验证错误

如果本次请求的参数未通过我们指定的验证规则，Laravel 会自动把用户重定向到先前的位置。另外，所有的验证错误会被自动**闪存至 session**

请注意在 `GET` 路由中，我们无需显式的将错误信息和视图绑定起来。这是因为 Lavarel 会检查在 Session 数据中的错误信息，然后如果对应的视图存在的话，自动将它们绑定起来。变量 `$errors` 会成为`Illuminate\Support\MessageBag` 的一个实例对象，绑定到视图，在视图中就可以获取到 $error 变量

#### AJAX 请求验证

当我们在 AJAX 的请求中使用 `validate` 方法时，Laravel 并不会生成一个重定向响应，而是会生成一个包含所有错误验证的 JSON 响应。这个 JSON 响应会发送一个 422 HTTP 状态码(我们可以在浏览器里面按 F12 来查看)

### 19、Blade 模板引擎

所有 Blade 视图文件都将被编译成原生的 PHP 代码并缓存起来，除非它被修改，否则不会重新编译

可以在 Blade 中显示任意的 PHP 代码：

```html
The current UNIX timestamp is {{ time() }}.
```

> Blade `{{ }}` 语法会自动调用 PHP `htmlspecialchars` 函数来避免 XSS 攻击

#### 当数据存在时输出

```html
{{ isset($name) ? $name : 'Default' }}
```

#### 显示未转义过的数据

在默认情况下，Blade 模板中的 `{{ }}` 表达式将会自动调用 PHP `htmlspecialchars` 函数(也就是把标签变成html实体)来转义数据以避免 XSS 的攻击。如果你不想你的数据被转义，你可以使用下面的语法：

```
{!! $name !!}
```

#### Blade & JavaScript 框架

由于很多 JavaScript 框架都使用花括号来表明所提供的表达式，所以你可以使用 `@` 符号来告知 Blade 渲染引擎你需要保留这个表达式原始形态，例如：

```html
<h1>Laravel</h1>
Hello, @{{ name }}.
```

`@` 符号最终会被 Blade 引擎剔除，并且 `{{ name }}` 表达式会被原样的保留下来，这样就允许你的 JavaScript 框架来使用它了

#### 循环变量

可以在循环内访问 `$loop` 变量。这个变量可以提供一些有用的信息，比如当前循环的索引，当前循环是不是首次迭代，又或者当前循环是不是最后一次迭代：

```html
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

在一个嵌套的循环中，可以通过使用 `$loop` 变量的 `parent` 属性来获取父循环中的 `$loop` 变量：

```
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

#### PHP

在模版中使用 Blade 提供的 `@php`指令来执行一段纯 PHP 代码：

```
@php
    //
@endphp
```

#### 堆栈

可以在其它视图或布局中为已经命名的堆栈中压入数据，例如把一个引入js代码的部分压入栈中：

```html
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

通过 `@stack` 命令中指定堆栈的名字来渲染整个堆栈：

```html
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

### 20、用户认证系统

Laravel 的认证组件由 `guards` 和 `providers` 组成，Guard 定义了用户在每个请求中如何实现认证，例如，Laravel 通过 `session` guard 来维护 Session 存储的状态和 Cookie

Provider 定义了如何从持久化存储中获取用户信息

#### 手动认证用户

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    public function authenticate()
    {
        if (Auth::attempt(['email' => $email, 'password' => $password])) {
            // Authentication passed...
            return redirect()->intended('dashboard');
        }
    }
}
```

`attempt` 方法会接受一个数组来作为第一个参数，这个数组的值可用来寻找数据库里的用户数据，所以在上面的例子中，用户通过 `email` 字段被取出，如果用户被找到了，**数据库里经过哈希**(也就是说，在存入数据库之前，要自己手动哈希)的密码将会与数组中哈希的 `password` (注意：数组中的密码 Laravel 会自动帮我们哈希，所以，我们不用自己哈希)值比对，如果两个值一样的话就会开启一个通过认证的 session 给用户

> `email` 不是一个一定要有的选项，它仅仅是被用来当作例子，你可以用任何字段，只要它在数据库的意义等同于用户名

### 21、数据库：入门

#### 运行原生 SQL 语句

使用 `DB` facade 来执行查询。 `DB` facade 提供了 `select` 、 `update` 、`insert` 、 `delete` 和 `statement` 的查询方法：

##### 运行 Select

```php
$users = DB::select('select * from users where active = ?', [1]);
```

传递到 `select` 方法的第一个参数是一个原生的 SQL 查询，而第二个参数则是传递的所有绑定到查询中的参数值。通常，这些都是 `where` 字句约束中的值。参数绑定可以避免 SQL 注入攻击

`select` 方法以数组的形式返回结果集，数组中的每一个结果都是一个PHP `StdClass` 对象，可以像下面这样访问结果值：

```php
foreach ($users as $user) {
    echo $user->name;
}
```

除了使用 `?` 来表示参数绑定外，你也可以使用命名绑定运行查找：

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

##### 运行 insert

运行 `Insert` 语句，你可以是使用 `DB` facade 的 `insert` 方法。该方法将原生SQL语句作为第一个参数，将参数绑定作为第二个参数：

```php
DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
```

##### 运行 Update

`update` 方法用于更新已经存在于数据库的记录：

```php
$affected = DB::update('update users set votes = 100 where name = ?', ['John']);
```

##### 运行 Delete

```php
$deleted = DB::delete('delete from users');
```

##### 运行一般声明

有些数据库没有返回值， 对于这种类型的操作，可以使用 `DB` facade 的 `statement` 方法：

```php
DB::statement('drop table users');
```

#### 数据库事务

在一个数据库事务中运行一连串操作，可以使用 `DB` facade 的 `transaction` 方法：

```php
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});
```

### 23、数据库请求构建器

Laravel 的查询构造器使用 PDO 参数绑定，来保护你的应用程序免受 SQL 注入的攻击。在绑定传入字符串前不需要清理它们

#### 获取结果

```php
 $users = DB::table('users')->get();
```

`get` 方法会返回一个 `Illuminate\Support\Collection` 结果，其中每个结果都是一个 PHP `StdClass` 对象的实例

##### 获取某列的值

获取一个包含两个字段值的集合，可以使用 `pluck` 方法

```php
$roles = DB::table('roles')->pluck('title', 'name');
```

将取出 roles 表中 title 、name 字段的集合

#### 聚合

查询构造器也支持各种聚合方法，如 `count`、 `max`、 `min`、 `avg` 和 `sum`。你可以在创建查询后调用其中的任意一个方法：

```php
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```

#### Joins

##### Inner Join 语法

如果要执行基本的「inner join」，可以在查询构造器实例上使用 `join` 方法。传递给 `join` 方法的第一个参数是要 join 数据表的名称，而其它参数则指定用来连接的字段约束

可以在单个查找中连接多个数据表：

```php
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
```

##### Left Join 语法

```php
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
```

##### Cross Join 语法

使用 `crossJoin` 方法和想要交叉连接的表名来做「交叉连接」。交叉连接通过第一个表和连接表生成一个笛卡尔积：

```php
$users = DB::table('sizes')
            ->crossJoin('colours')
            ->get();
```

#### Unions

合并 两个查询：

```php
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
```

#### Where 子句

##### 简单的 Where 子句

基本的 `where` 方法需要3个参数。第一个参数是字段的名称。第二个参数是运算符，它可以是数据库所支持的任何运算符，第三个参数是要对字段进行评估的值：

```php
$users = DB::table('users')->where('votes', '=', 100)->get();
```

```php
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

##### Or 语法

```php
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```

#### 它 Where 子句

##### 验证字段的值介于两个值之间(一个连续的范围)：

```php
$users = DB::table('users')
                    ->whereBetween('votes', [1, 100])->get();
```

##### 验证字段的值 **不** 在两个值之间：

```php
$users = DB::table('users')
                    ->whereNotBetween('votes', [1, 100])
                    ->get();
```

##### 验证字段的值包含在指定的数组内(离散的范围)：

```php
$users = DB::table('users')
                    ->whereIn('id', [1, 2, 3])
                    ->get();
```

##### 验证字段的值不包含在指定的数组内：

```php
$users = DB::table('users')
                    ->whereNotIn('id', [1, 2, 3])
                    ->get();
```

#### Ordering, Grouping, Limit 及 Offset

##### orderBy

```php
$users = DB::table('users')
                ->orderBy('name', 'desc')
                ->get();
```

##### latest / oldest

**依据日期**对查询结果排序。默认查询结果将**依据 `created_at` 列**。或者,你可以使用字段名称排序：

```php
$user = DB::table('users')
                ->latest()
                ->first();
```

### 24、数据库迁移 Migrations

Laravel 的 `Schema` facade对所有 Laravel 支持的数据库系统提供了创建和操作数据表的相应支持

#### 创建迁移表

```
php artisan make:migration create_articles_table
```

#### 运行迁移

使用 `migrate` Artisan 命令，来运行所有未运行过的迁移：

```
php artisan migrate
```

#### 回滚迁移

回滚最后一次迁移(对上一次执行的「**批量**」迁移回滚，其中可能包括多个迁移文件)：

```
php artisan migrate:rollback
```

回滚应用程序中的所有迁移：

```
php artisan migrate:reset
```

重新创建整个数据库(回滚数据库的所有迁移还会接着运行 `migrate` 命令)：

```
php artisan migrate:refresh
```

#### 外键约束

用于在数据库层中的强制引用完整性。例如，让我们定义一个有 `user_id` 字段的 `posts` 数据表，`user_id` 引用了 `users` 数据表的 `id` 字段：

```php
Schema::table('posts', function (Blueprint $table) {
    $table->integer('user_id')->unsigned();

    $table->foreign('user_id')->references('id')->on('users');
});
```

#### 开启和关闭外键约束：

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();
```

### 25、数据库之：数据填充

seed 类轻松地为数据库填充测试数据。所有的 seed 类都存放在 `database/seeds` 目录下。你可以任意为 seed 类命名，但是应该遵守类似 `UsersTableSeeder` 的命名规范。Laravel 默认定义了一个 `DatabaseSeeder`类。可以在这个类中使用 `call` 方法来运行其它的 seed 类来控制数据填充的顺序

#### 生成一个 Seeder

```
php artisan make:seeder UsersTableSeeder
```

一个 seeder 类只包含一个默认方法：`run`。这个方法在 `db:seed` Artisan 命令被调用时执行。在 `run` 方法里你可以为数据库添加任何数据。你也可以用**查询语句构造器**或**Eloquent 模型工厂**来手动添加数据

#### 调用其他 Seeders

在 `DatabaseSeeder` 类中，你可以使用 `call` 方法来运行其他的 seed 类。为避免单个 seeder 类过大，可使用`call` 方法将数据填充拆分成多个文件。只需简单传递要运行的 seeder 类名称即可：

```php
public function run()
{
    $this->call(UsersTableSeeder::class);
    $this->call(PostsTableSeeder::class);
    $this->call(CommentsTableSeeder::class);
}
```

#### 运行 Seeders

在默认情况下，`db:seed` 命令将运行 `DatabaseSeeder` 类(可以用来调用其他填充类)：

```
php artisan db:seed
```

指定运行一个特定的 seeder 类：

```
php artisan db:seed --class=UsersTableSeeder
```

### 26、Eloquent: 入门

#### 创建模型实例

```
php artisan make:model User
```

创建模型实例同时顺便生成一个数据库迁移：

```
php artisan make:model User --migration
```

#### 数据库连接

默认情况下，所有的 Eloquent 模型会使用应用程序中默认的数据库连接设置。如果你想为模型指定不同的连接，可以使用 `$connection` 属性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    protected $connection = 'connection-name';
}
```

#### 取回模型

##### 返回在模型数据表中的所有结果(包括关联模型也会被全部取出来)

```php
$flights = App\Flight::all();
```

##### 增加额外的限制

由于每个 Eloquent 模型都可以当作一个查询构造器，所以**可以在查询中增加规则，然后使用 `get` 方法来获取结果**：

```php
$flights = App\Flight::where('active', 1)
               ->orderBy('name', 'desc')
               ->take(10)
               ->get();
```

**由于 Eloquent 模型是查询构造器 ，所以可以在 Eloquent 查询中使用这其中的任何方法**

##### 分块结果

获取一个 Eloquent 模型的「分块」，并将它们送到指定的 `闭包 (Closure)` 中进行处理：

```php
Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
```

在处理大量结果时，使用 `chunk` 方法可节省内存：

传递到方法的第一个参数表示每次「分块」时你希望接收的数据数量。**闭包则作为第二个参数传递，它将会在每次从数据取出分块时被调用**

##### 使用游标

使用游标来遍历数据库数据，一次只执行单个查询：

```php
foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
    //
}
```

在处理大数据量请求时 `cursor` 方法可以大幅度减少内存的使用

#### 取回单个模型

##### 通过 `find` 和 `first` 方法来取回单条记录

```php
// 通过主键取回一个模型...
$flight = App\Flight::find(1);

// 取回符合查询限制的第一个模型 ...
$flight = App\Flight::where('legs', '>', 100)->first();
```

这些方法返回的是单个模型的实例，而不是返回模型的集合

##### 用主键的集合为参数调用`find`方法，它将返回符合条件的集合：

```php
$flights = App\Flight::find([1, 2, 3]);
```

#### 添加和更新模型

##### 基本添加

```php
$flight = new Flight;
$flight->name = 'xiaohuanghuang';
$flight->save();
```

##### 基本更新

要更新模型，则须**先取出模型**，再设置任何你希望更新的属性，接着调用 `save` 方法：

```php
$flight = App\Flight::find(1);
$flight->name = 'xiaozouzou';
$flight->save();
```

##### 批量更新

**所有** `active` 为1并且 `destination` 为 `San Diego` 的航班，都将会被标识为延迟：

```php
App\Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);
```

##### 批量赋值

使用 `create` 方法通过一行代码来保存一个新模型。此时，需要先在对应的模型上定义一个 `fillable` 或 `guarded` 属性，因为所有的 Eloquent 模型都针对批量赋值（Mass-Assignment）做了保护

当用户通过 HTTP 请求传入了非预期的参数，并借助这些参数更改了数据库中你并不打算要更改的字段，这时就会出现批量赋值（Mass-Assignment）漏洞。例如，恶意用户可能会通过 HTTP 请求发送 `is_admin` 参数，然后对应到你模型的 `create` 方法，此操作能让该用户把自己升级为一个管理者

```php
$flight = App\Flight::create(['name' => 'Flight 10']);
```

**`create` 方法将返回已经被保存的模型实例(相当于new App\Flight())**

执行完create方法之后，记录就保存在数据库里面了

#### 删除模型

##### 先找到这个模型，然后删除

```php
$flight = App\Flight::find(1);
$flight->delete();
```

##### 通过键来直接删除现有的模型

```php
App\Flight::destroy(1);
```

```php
App\Flight::destroy([1, 2, 3]);
```

##### 通过查询来删除模型

```php
$deletedRows = App\Flight::where('legs', '>', 100)->delete();
```

#### 软删除

当模型被软删除时，它们并不会真的从数据库中被移除。而是会在模型上设置一个 `deleted_at` 属性并将其添加到数据库

#### 事件

Eloquent 模型会触发许多事件，在模型的生命周期的多个时间点进行监控

每当有特定的模型类在数据库保存或更新时，执行对应的事件的代码

当一个新模型被**初次保存**将会触发 `creating` 以及 `created` 事件。如果一个模型**已经存在于数据库**且调用了`save` 方法，将会触发 `updating` 和 `updated` 事件

(以上两种情况下都会触发 `saving` 和 `saved` 事件)

开始前，在 Eloquent 模型上定义一个 `$events` 属性，将 Eloquent 模型的生命周期的多个点映射到你的服务提供者

```php
<?php

namespace App;

use App\Events\UserSaved;
use App\Events\UserDeleted;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * 模型的时间映射。
     *
     * @var array
     */
    protected $events = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```

##### 观察者

如果你在一个给定的模型中监听许多事件，您**可以使用观察者将所有监听器变成一个类**。观察者类里的方法名应该反映Eloquent想监听的事件。 每种方法接收 model 作为其唯一的参数。 Laravel不包括观察者默认目录，所以你可以创建任何你喜欢你的目录来存放：

```php
<?php

namespace App\Observers;

use App\User;

class UserObserver
{
    /**
     * 监听用户创建的事件。
     *
     * @param  User  $user
     * @return void
     */
    public function created(User $user)
    {
        //
    }

    /**
     * 监听用户删除事件。
     *
     * @param  User  $user
     * @return void
     */
    public function deleting(User $user)
    {
        //
    }
}
```

**注册这个观察者**

要注册一个观察者，需要用模型中的`observe`方法去观察。可以在任意一个服务提供商的`boot`方法中注册观察者，例如：

```php
<?php

namespace App\Providers;

use App\User;
use App\Observers\UserObserver;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 运行所有应用.
     *
     * @return void
     */
    public function boot()
    {
        User::observe(UserObserver::class);
    }

    /**
     * 注册服务提供.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

### 27、Eloquent: 关联

#### 一对一

一个用户有一个手机，所以在User模型中这样写phone方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 获取与用户关联的电话号码
     */
    public function phone()
    {
        return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
    }
}
```

使用 Eloquent 的动态属性来获取关联记录：

```php
$phone = User::find(1)->phone;
```

#### 反向关联

每个手机属于一个用户：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Phone extends Model
{
    /**
     * 获取拥有该电话的用户模型。
     */
    public function user()
    {
        return $this->belongsTo('App\User', 'foreign_key', 'other_key');
    }
}
```

**虽然此时也是一对一的，但是，在user方法里面是用不到hasOne方法的，因为，在User模型里面是没有phone的外键**

**注意，这里的外键始终都是指phone模型里的user_id，而不是指第一个参数的**

#### 一对多

一篇文章有多个评论：

```php
public function comments()
{
	return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
}
```

调用`comments` 方法然后在该方法后面链式调用查询条件来获取评论：

```php
$comments = App\Post::find(1)->comments()->where('title', 'foo')->first();
```

#### 多对多

一个用户可能拥有多种身份，而一种身份能同时被多个用户拥有：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 属于该用户的身份。
     */
    public function roles()
    {
        return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');
    }
}
```

操作多对多关联需要一张关系表

在`belongsToMany`函数里面，第二个参数是关系表的名称，第三个参数是定义在关联中的模型外键(因为是在User中定义的关联，所以第三个参数是相对于user的外键--user_id)，第四个参数是要合并的模型外键名称(也就是剩下的那个外键)

获取数据：

```php
$roles = App\User::find(1)->roles;
```

访问关系表的数据：

```php
$user = App\User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

取出的每个 `Role` 模型对象，都会被自动赋予 `pivot` 属性。此属性代表中间表的模型，它可以像其它的 Eloquent 模型一样被使用

#### 查找关联

所有类型的 Eloquent 关联也提供了查询语句构造器的功能

例如，可以查找 `posts` 关联并增加额外的条件至关联：

```php
$user = App\User::find(1);

$user->posts()->where('active', 1)->get();
```

查找用户1的所有文章，并且文章的`active`字段值为1

##### 关联方法与动态属性

**如果不需要增加额外的条件**至 Eloquent 的关联查找，则可以简单的像访问属性一样来访问关联

```php
$user = App\User::find(1);

foreach ($user->posts as $post) {
    //
}
```

**动态属性**是「延迟加载」的，意味着它们**只会在被访问的时候才加载关联数据**。正因为如此，开发者**通常需要使用预加载来预先加载关联数据，关联数据将会在模型加载后被访问**。预加载能有效减少你的 SQL 查询语句

##### 查找关联是否存在

获取博客中那些至少拥有一条评论的文章：

```php
// 获取那些至少拥有一条评论的文章...
$posts = App\Post::has('comments')->get();
```

```php
// 获取所有至少有三条评论的文章...
$posts = Post::has('comments', '>=', 3)->get();
```

##### 关联数据计数

```php
$posts = App\Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

还可以像在查询语句中添加约束一样，获取多重关联的「计数」：

```php
$posts = Post::withCount(['votes', 'comments' => function ($query) {
    $query->where('content', 'like', 'foo%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

#### 预加载

当通过属性访问 Eloquent 关联时，该关联数据会被「延迟加载」。意味着该关联数据只有在你使用属性访问它时才会被加载

预加载避免了 N + 1 查找的问题。要说明 N + 1 查找的问题，可试想一个关联到 `Author` 的 `Book` 模型：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    /**
     * 获取编写该书的作者。
     */
    public function author()
    {
        return $this->belongsTo('App\Author');
    }
}
```

获取所有书籍及其作者的数据：

```php
$books = App\Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

**上方的循环会运行一次查找并取回所有数据表上的书籍，接着每本书会运行一次查找作者的操作。因此，若存在着 25 本书，则循环就会执行 26 次查找：1 次是查找所有书籍，其它 25 次则是在查找每本书的作者**

可以使用预加载来将查找的操作减少至 2 次

```php
$books = App\Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

该操作则只会执行两条 SQL 语句：

```sql
select * from books
select * from authors where id in (1, 2, 3, 4, 5, ...)
```

#### 写入数据到关联模型

##### Save 方法

将新的 `Comment` 模型写入至 `Post` 模型。可以直接使用关联的 `save` 方法来写入 `Comment`：

```php
$comment = new App\Comment(['message' => 'A new comment.']);
$post = App\Post::find(1);
$post->comments()->save($comment);
```

save 方法会**自动**在新的 `Comment` 模型中增加正确的 `post_id` 值，这样就不用手动的(自己写代码)去设置这个刚new出来的Comment模型的post_id了

### 28、Eloquent: 序列化

#### 序列化模型 & 集合

##### 序列化成数组

```php
$user = App\User::with('roles')->first();
return $user->toArray();
```

将整个集合(`Illuminate\Database\Eloquent\Collection`对象的实例)转换成数组：

```php
$users = App\User::all();
return $users->toArray();
```

##### 序列化成 JSON

```php
$user = App\User::find(1);
return $user->toJson();
```

#### 隐藏来自 JSON 的属性

在模型中增加 `$hidden`属性定义来实现：

```php
protected $hidden = ['password'];
```

也可以使用 `visible` 属性来定义应该包含在你的模型数组和 JSON 表示中的属性白名单。白名单外的其他属性将隐藏，不会出现在转换后的数组或 JSON 中：

```php
protected $visible = ['first_name', 'last_name'];
```





























































































































