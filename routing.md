# HTTP Routing

- [基础路由](#basic-routing)
- [路由参数](#route-parameters)
    - [必要参数](#required-parameters)
    - [可选的参数](#parameters-optional-parameters)
    - [参数的正则约束](#parameters-regular-expression-constraints)
- [命名路由](#named-routes)
- [路由组](#route-groups)
    - [中间件](#route-group-middleware)
    - [命名空间](#route-group-namespaces)
    - [子域名路由](#route-group-sub-domain-routing)
    - [路由前缀](#route-group-prefixes)
- [CSRF 保护](#csrf-protection)
    - [前言](#csrf-introduction)
    - [排除 URIs](#csrf-excluding-uris)
    - [X-CSRF-Token](#csrf-x-csrf-token)
    - [X-XSRF-Token](#csrf-x-xsrf-token)
- [路由模型绑定](#route-model-binding)
- [虚假的 form 方法](#form-method-spoofing)
- [访问当前路由](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基础路由

一般所有的路由都被定义在 `app/Http/routes.php` 文件中，其中的内容会被框架自动加载，大多数的基础路由都可以通过简单的传递资源表述地址和 `Closure` 闭包函数来进行定义。

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 基础路由文件

`routes.php` 路由文件是在 `app\Providers\RouteServiceProvider.php` 中被加载，在被加载的同时使用了 `web` 中间件组，这个中间件组被定义在 `app\Http\Kernel.php` 的 `$middlewareGroups` 变量中，该中间件组提供了 cookie 加密、cookie 响应、session 启用、自动注入 error session 到视图、csrf 保护的功能。你的应用程序的大多数路由都应该定义在这个路由文件中。

#### 可用的路由方法

路由允许你在注册路由时使用任意的 HTTP 请求方法：

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有时你可能需要在注册路由时允许多种请求方式，这个时候你可能需要使用 `match` 方法，当然你可以使用 `any` 方法来允许任意的请求方式：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

<a name="route-parameters"></a>
## 带参数的路由

<a name="required-parameters"></a>
### 必要参数的路由

有时候你可能需要捕获路由中的某个片段，比如说用户的 id，那么你就可以定义带参数的路由：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

可能你需要获取到路由中的多个参数，那么你可以这么做：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由参数就是定义在 "{}" 里的，每当路由被访问到，相关注册的参数就会被按序的传递进闭包函数里。

> **注意** 路由参数不能包含 `-` 字符串。你应该使用 `_` 来取代它。

<a name="parameters-optional-parameters"></a>
### 可选的路由参数

有时候你可能需要定义一个带参数的路由，但是这个参数可有可无，没有的话，可以给予一个默认值，那么这时候就需要用 `{name?}` 的方式来定义可选路由了：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 用正则表达式约束参数

你可以使用正则表单时去约束你的参数，这里你可以使用 `where` 方法，当参数与约束规则不匹配时就不会匹配到该路由：

    Route::get('user/{name}', function ($name) {
        //
    })
    ->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })
    ->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })
    ->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### 全局的约束表达式

你可以定义全局的约束表达式，这样所有匹配到的路由参数都会受到相同的约束条件，你可以使用 `pattern` 方法来进行约束，你需要在路由服务提供者 `RouteServiceProvider`文件中的 `boot` 方法里定义这些约束。

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @param  \Illuminate\Routing\Router  $router
     * @return void
     */
    public function boot(Router $router)
    {
        $router->pattern('id', '[0-9]+');

        parent::boot($router);
    }

当约束表达式定义完成，它会将约束自动的传递到所有使用相同参数名称的路由中：

    Route::get('user/{id}', function ($id) {
        // Only called if {id} is numeric.
    });

<a name="named-routes"></a>
## 命名路由

命名路由可以方便的生成资源表述地址，可以方便的将路由重定向到指定的路由地址。你可以在注册路由时，传递第二个参数为数组参数，并添加 `as` 键进行命名：

    Route::get('user/profile', ['as' => 'profile', function () {
        //
    }]);

当然你也可以命名使用控制器行为的路由：

    Route::get('user/profile', [
        'as' => 'profile', 'uses' => 'UserController@showProfile'
    ]);

另外，有时候数组写起来比较麻烦，我们还可以使用 `name` 方法来命名路由，当然，它是支持链式调用的:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 路由组 & 路由组命名

如果你使用了 [路由组](#route-groups)，你可能想在整个组中增加一个组命名前缀，当然，Laravel 是支持这么做的：

    Route::group(['as' => 'admin::'], function () {
        Route::get('dashboard', ['as' => 'dashboard', function () {
            // Route named "admin::dashboard"
        }]);
    });

#### 使用命名路由生成 URLs

当你为一个路由命名之后，你可以使用全局帮助方法 `route` 来生成路由 URLs，也可以用来作为重定向时生成重定向地址：

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

如果你是对一个参数路由进行命名，那么你可以在使用 `route` 方法获取路由时传递第二个参数，该参数是一个数组，你所传递的参数会正确的按索引匹配到相应的位置，如果参数索引与路由不一致，则会优先匹配相同参数存放到相应位置，不一致的会按序存入：

    Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
        //
    }]);

    $url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## 路由组

路由组的使用可以方便的共享路由的一些能力，比如共享中间件，或者命名空间，这样就不必每一个路由都要单独去定义一次相同的中间件或命名空间。路由组使用 `Route::group` 方法定义，并共享首个参数中的能力，该参数应是一个数组。

为了更好的了解路由组，我们将通过一些常用示例来展示其特性。

<a name="route-group-middleware"></a>
### 共享中间件

你可以使用 `middleware` 索引来建立共享中间件，这样路由组内定义的路由都会在匹配时引入该中间件：

    Route::group(['middleware' => 'auth'], function () {
        Route::get('/', function ()    {
            // Uses Auth Middleware
        });

        Route::get('user/profile', function () {
            // Uses Auth Middleware
        });
    });

<a name="route-group-namespaces"></a>
### 命名空间

路由组的另外一个常用的示例就是在控制器中共享命名空间，这个时候就需要用到 `namespace` 索引了：

    Route::group(['namespace' => 'Admin'], function()
    {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace

        Route::group(['namespace' => 'User'], function() {
            // Controllers Within The "App\Http\Controllers\Admin\User" Namespace
        });
    });

还记得吗，默认的，在 `RouteServiceProvider` 引入你的 `routes.php` 文件时已经将其包含在一个路由组中了，并给这个路由组分配了命名空间 `App\Http\Controllers`，这样，在该文件中定义的控制器路由，可以不使用 `App\Http\Controllers` 前缀。我们无需重复填写前缀，只需要指定基于该命名空间之后的部分就可以了。

<a name="route-group-sub-domain-routing"></a>
### 子域名路由组

子域名路由组可以做域名匹配路由，也就是说其可以约束域名做匹配，如果不是匹配到的域名则不会匹配到该组内的路由。子域名也可以像路由 URIs 一样分配参数，这允许你捕获子域名路由的一部分而用于路由或者控制器中。你可以通过在路由组的属性中指定 `domain` 索引来定义一个子域名路由组：

    Route::group(['domain' => '{account}.myapp.com'], function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### 路由组前缀

使用 `prefix` 索引可以在路由组中定义路由前缀，这样在该组内的路由都会自动使用该前缀，比如，你可能希望给路由组中的路由全都增加一个 `admin` 前缀:

    Route::group(['prefix' => 'admin'], function () {
        Route::get('users', function ()    {
            // Matches The "/admin/users" URL
        });
    });

当然你也可以在路由前缀中使用参数:

    Route::group(['prefix' => 'accounts/{account_id}'], function () {
        Route::get('detail', function ($accountId)    {
            // Matches The "/accounts/{account_id}/detail" URL
        });
    });

<a name="csrf-protection"></a>
## CSRF 保护

<a name="csrf-introduction"></a>
### 前言

对于来自 [cross-site request forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 的攻击，Laravel 可以轻松的做出保护。跨站请求伪造是一种恶意的攻击行为，它可以绕过用户的授权去执行未授权的行为。

Laravel 会指定的为每个活跃用户生成一个 CSRF "token"，而这个 token 就是用来验证请求是不是真实用户授权的。

无论何时，当你在应用中定义 HTML 表单时，你都应该引入隐藏的 CSRF token 字段到你的表单中，这样 CSRF 保护中间件才会允许该请求通过。你可以使用 `csrf_field` 帮助函数来生成一个包含了 CSRF token 的隐藏 input 字段:
Anytime you define a HTML form in your application, you should include a hidden CSRF token field in the form so that the CSRF protection middleware will be able to validate the request. To generate a hidden input field `_token` containing the CSRF token, you may use the `csrf_field` helper function:

    // Vanilla PHP
    <?php echo csrf_field(); ?>

    // Blade Template Syntax
    {{ csrf_field() }}

`csrf_field` 帮助方法会生成以下 HTML 节点：

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

你并不需要去手动的验证这些能够修改状态的 HTTP 请求，比如 POST，PUT，或者 DELETE 请求，`VerifyCsrfToken` [middleware](/docs/{{language}}/{{version}}/middleware) 已经自动的处理这些请求了，它会自动的根据请求中的 token 与会话中存储的 token 进行匹配验证。

<a name="csrf-excluding-uris"></a>
### 从 CSRF 保护中排除某些 URLs

有时候我们可能需要排除个别 URLs，让他不受 CSRF 的保护，因为可能我们需要对接一些第三方系统，这个时候系统之间的通讯是不应该使用 CSRF 机制去验证的。比如你使用了支付宝的即时付款机制，那么在用户付款的时候服务器与服务器之间会有一个回馈机制，用于支付宝通知我们的应用用户已经完成付款，这个时候我们可能需要排除 CSRF 对反馈路由的保护。

你可以在引入 `web` 中间件的路由组之外定义独立的 URLs 路由，或者你可以在 `VerifyCsrfToken` 中的 `$except` 变量中追加 URIs 来避免 CSRF 的保护:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

    class VerifyCsrfToken extends BaseVerifier
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
### X-CSRF-TOKEN

Laravel 的 `VerifyCsrfToken` 中间件不仅允许通过表单中验证 `_token` 参数，它也允许通过请求头进行 CSRF 验证，它会验证请求头中的 `X-CSRF-TOKEN` 的值是否与会话中的 token 值匹配，所以你也可以这么做，存储 token 到 `meta` 标签中:

    <meta name="csrf-token" content="{{ csrf_token() }}">

当你定义了 `meta` 标签之后，你可以指导像 jQuery 类似的框架中添加 token 到所有请求的头部，这种方式可以非常简单方便的为基于 AJAX 类的应用提供 CSRF 保护：

    $.ajaxSetup({
            headers: {
                'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
            }
    });

<a name="csrf-x-xsrf-token"></a>
### X-XSRF-TOKEN

Laravel 也会存储 CSRF token 到 `XSRF-TOKEN` cookie 中。你也可以在每次的请求头里引入 `X-XSRF-TOKEN` 并将其设置为 `XSRF-TOKEN` 的 cookie 值来进行 CSRF 验证。有一些 JavaScript 框架，像 Angular，会自动的为你做这些。你不太可能需要手动的使用这个值。
    
<a name="route-model-binding"></a>
## 绑定模型到路由

绑定模型到路由提供了一种便利的方式去注入模型实例到路由中，比如，你可以通过传递 id，来注入匹配 id 的用户的整个模型实例到路由中。

### 隐式绑定

Laravel 会自动的解析定义在路由或者控制器中具有类型提示的 `Eloquent` 模型，根据变量名与参数名匹配，比如 `api/users/{user}` 匹配 `{$user}` 变量：

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

在上面的例子中，具有类型提示的 Eloquent $user 变量匹配 URL 中 `{user}` 片段，所以 Laravel 会自动的根据请求所匹配的用户 ID 来注入相应的用户模型实例。

如果所匹配的模型在数据库中不存在，那么自动的生成一个 404 的 HTTP 响应。

#### 自定义索引键

如果你不想用数据库中的 id 主键去绑定模型到路由，那么你可以在你的 Eloquent 模型中重载 `getRouteKeyName` 方法来自定义索引键：

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

### 显示绑定

如果需要注册一种显式的模型绑定到路由，你需要使用路由的 `model` 方法手动的指定类与参数的映射。你应该在 `RouteServiceProvider::boot` 方法中去做这些绑定：

#### 绑定一个参数到一个路由

    public function boot(Router $router)
    {
        parent::boot($router);

        $router->model('user', 'App\User');
    }

接着，定义一个包含了 `{user}` 参数的路由：

    $router->get('profile/{user}', function(App\User $user) {
        //
    });

由于我们已经绑定了 `{user}` 参数到 `App\User` 模型中，所以，一个 `User` 实例将会被自动的注入到路由中。那么一个路径到 `profile/1` 的请求将会在路由中自动的注入 ID 为 1 的 `User` 实例。

如果所匹配的模型实例在数据库中不存在，那么路由将会自动的生成一个 404 HTTP 响应。

#### 自定义绑定解析逻辑

如果你想自定义解析返回实例的逻辑，那么你需要使用 `Route::bind` 方法，该方法第一个参数为绑定 URL 的参数名，第二个参数为 `Closure`，闭包中会接收一个参数，这个参数是 URL 中对应参数片段的值，你应该在闭包中返回解析后的实例：

    $router->bind('user', function ($value) {
        return App\User::where('name', $value)->first();
    });

#### 自定义找不到实例时的行为

如果你想要自定义找不到匹配的实例时的行为，你可以传递 `model` 方法第三个参数，该参数是一个闭包函数，它将在无法匹配到实例时执行：

    $router->model('user', 'App\User', function () {
        throw new NotFoundHttpException;
    });

<a name="form-method-spoofing"></a>
## 表单提交方式模拟

由于 HTML 表单不能支持像 `PUT`，`PATCH` 或者 `DELETE` 的请求方式。所以，当需要访问类似的路由时，你需要在表单中增加一个隐藏的字段 `_method`，用来表明你真实的表单请求方式:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Laravel 也提供了便捷的辅助方法 `method_filed` 来生成隐藏的 `_method` 节点：

    <?php echo method_field('PUT'); ?>

当然，使用 Blade [模板引擎](/docs/{{language}}/{{version}}/blade) 你可以这么做:

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## 访问当前路由

`Route::current()` 方法会返回一个完整的 `Illuminate\Routing\Route` 实例，它允许你在当前路由中处理相关请求：

    $route = Route::current();

    $name = $route->getName();

    $actionName = $route->getActionName();

当然你也可以使用 `Route` 假面的 `currentRouteName` 和 `currentRouteAction` 方法来访问当前路由的名称和行为:

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

请参考 API 文档 [underlying class of the Route facade](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 和 [Route instance](http://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) 浏览所有可访问的方法。
