# 路由

- [基础路由](#basic-routing)
- [路由参数](#route-parameters)
    - [必要参数](#required-parameters)
    - [可选参数](#parameters-optional-parameters)
- [命名路由](#named-routes)
- [路由组](#route-groups)
    - [中间件](#route-group-middleware)
    - [命名空间](#route-group-namespaces)
    - [子域名路由](#route-group-sub-domain-routing)
    - [路由前缀](#route-group-prefixes)
- [路由模型绑定](#route-model-binding)
    - [隐式绑定](#implicit-binding)
    - [显示绑定](#explicit-binding)
- [虚假的表单方法](#form-method-spoofing)
- [访问当前路由](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基础路由

大多数的基础路由都可以通过简单的传递资源表述地址和 `Closure` 闭包函数来进行定义。

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 基础路由文件

Laravel 中所有的路由都被定义在路由文件中，这些文件被存放在 `routes` 目录下。这个目录下的文件会被框架自动的加载。其中 `routes/web.php` 文件里定义了一些路由为你的 web 接口提供服务。这些路由被分配了 `web` 中间件组，这个中间件组提供了一些如会话状态和 CSRF 包含的特性。而 `routes/api.php` 文件服务于无状态的请求，并且它被分配了 `api` 中间件组。

对于大多数应用来说，你可以直接开始在 `routes/web.php` 文件中定义路由。

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

#### CSRF 保护

任意指向 `POST`，`PUT`，或者 `DELETE` 类型并被定义在 `web` 路由文件中的路由的 HTML 表单都应该被引入 CSRF token 节点。不然的话，这请求将会被拒绝。你可以阅读 [CSRF 文档](/docs/{{language}}/{{version}}/csrf) 来了解更多关于 CSRF 保护的信息。

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

<a name="route-parameters"></a>
## 路由参数

<a name="required-parameters"></a>
### 必要参数

有时候你可能需要捕获路由中的某个片段，比如说用户的 ID，那么你就可以定义带参数的路由：

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

可能你需要获取到路由中的多个参数，那么你可以这么做：

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

路由参数就是定义在 "{}" 里的，每当路由被访问到，相关注册的参数就会被按序的传递进闭包函数里。路由参数不能包含 `-` 字符串。你应该使用 `_` 来取代它。

<a name="parameters-optional-parameters"></a>
### 可选参数

有时候你可能需要定义一个带参数的路由，但是这个参数可有可无，没有的话，可以给予一个默认值，那么这时候就需要用 `{name?}` 的方式来定义可选路由了：

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="named-routes"></a>
## Named Routes

命名路由可以方便的生成资源表述地址，可以方便的将路由重定向到指定的路由地址。你可以在注册路由时，通过链式调用 `name` 方法来为路由指定一个名称：

    Route::get('user/profile', function () {
        //
    })->name('profile');

当然你也可以命名使用控制器行为的路由：

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### 使用命名路由生成 URLs

当你为一个路由命名之后，你可以使用全局帮助方法 `route` 来生成路由 URLs，也可以用来作为重定向时生成重定向地址：

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

如果你是对一个参数路由进行命名，那么你可以在使用 `route` 方法生成路由地址时传递第二个参数，该参数是一个数组，你所传递的参数会正确的按索引匹配到相应的位置，如果参数索引与路由不一致，则会优先匹配相同参数存放到相应位置，不一致的会按序存入：

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## 路由组

路由组的使用可以方便的共享路由的一些能力，比如共享中间件，或者命名空间，这样就不必每一个路由都要单独去定义一次相同的中间件或命名空间。路由组使用 `Route::group` 方法定义，并共享首个参数中的能力，该参数应是一个数组。

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

    Route::group(['namespace' => 'Admin'], function() {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

还记得吗，默认的，在 `RouteServiceProvider` 引入你的路由文件时已经将其包含在一个路由组中了，并给这个路由组分配了命名空间 `App\Http\Controllers`，这样，在该文件中定义的控制器路由，可以不使用 `App\Http\Controllers` 前缀。我们无需重复填写前缀，只需要指定基于该命名空间之后的部分就可以了。

<a name="route-group-sub-domain-routing"></a>
### 子域名路由组

路由组也可以做子域名匹配路由，也就是说其可以约束域名做匹配，如果不是匹配到的域名则不会匹配到该组内的路由。子域名也可以像路由 URIs 一样分配参数，这允许你捕获子域名路由的一部分而用于路由或者控制器中。你可以通过在路由组的属性中指定 `domain` 索引来定义一个子域名路由组：

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

<a name="route-model-binding"></a>
## 路由模型绑定

当注入模型的 ID 到路由或控制器动作时，你经常会需要查询检索出相应 ID 的模型。Laravel 路由模型绑定提供了一种便利的方式去注入模型实例到路由中，比如，你可以通过传递 ID，来注入匹配 ID 的用户的整个模型实例到路由中。

<a name="implicit-binding"></a>
### 隐式绑定

Laravel 会自动的解析定义在路由或者控制器中具有类型提示的 `Eloquent` 模型，它会根据变量名与参数名匹配，比如 `api/users/{user}` 匹配 `$user` 变量：

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

在上面的例子中，具有类型提示的 Eloquent `$user` 变量匹配 URL 中 `{user}` 片段，所以 Laravel 会自动的根据请求所匹配的用户 ID 来注入相应的用户模型实例。如果所匹配的模型在数据库中不存在，那么自动的生成一个 404 的 HTTP 响应。

#### 自定义索引键

如果你不想用数据库中的 `id` 主键去绑定模型到路由，那么你可以在你的 Eloquent 模型中重载 `getRouteKeyName` 方法来自定义索引键：

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### 显示绑定

如果需要注册一种显式的模型绑定到路由，你需要使用路由的 `model` 方法手动的指定类与参数的映射。你应该在 `RouteServiceProvider::boot` 方法中去做这些绑定：

    public function boot()
    {
        parent::boot();

        Route::model('user', 'App\User');
    }

接着，定义一个包含了 `{user}` 参数的路由：

    $router->get('profile/{user}', function(App\User $user) {
        //
    });

由于我们已经绑定了 `{user}` 参数到 `App\User` 模型中，所以，一个 `User` 实例将会被自动的注入到路由中。那么一个路径到 `profile/1` 的请求将会在路由中自动的注入 ID 为 1 的 `User` 实例。

如果所匹配的模型实例在数据库中不存在，那么路由将会自动的生成一个 404 HTTP 响应。

#### 自定义解析逻辑

如果你想自定义解析返回实例的逻辑，那么你需要使用 `Route::bind` 方法，该方法第一个参数为绑定 URL 的参数名，第二个参数为 `Closure`，闭包中会接收一个参数，这个参数是 URL 中对应参数片段的值，你应该在闭包中返回解析后的实例：

    $router->bind('user', function ($value) {
        return App\User::where('name', $value)->first();
    });

<a name="form-method-spoofing"></a>
## 虚假的表单方法

由于 HTML 表单不能支持像 `PUT`，`PATCH` 或者 `DELETE` 的请求方式。所以，当需要访问类似的路由时，你需要在表单中增加一个隐藏的字段 `_method`，用来表明你真实的表单请求方式:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

Laravel 也提供了便捷的辅助方法 `method_filed` 来生成隐藏的 `_method` 节点：

    {{ method_field('PUT') }}

<a name="accessing-the-current-route"></a>
## 访问当前路由

你可以使用 `Route` 假面的 `current`，`currentRouteName` 和 `currentRouteAction` 方法来访问当前处理请求的路由的详细信息:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

请参考 API 文档 [underlying class of the Route facade](/api/{{version}}/Illuminate/Routing/Router.html) 和 [Route instance](/api/{{version}}/Illuminate/Routing/Route.html) 浏览所有可访问的方法。
