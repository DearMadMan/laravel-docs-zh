# HTTP 控制器

- [控制器](#introduction)
- [基础控制器](#basic-controllers)
- [控制器中间件](#controller-middleware)
- [RESTful 资源控制器](#restful-resource-controllers)
    - [部分资源路由](#restful-partial-resource-routes)
    - [命名资源路由](#restful-naming-resource-routes)
    - [命名资源路由参数](#restful-naming-resource-route-parameters)
    - [意外的行为](#restful-supplementing-resource-controllers)
- [依赖注入 & 控制器](#dependency-injection-and-controllers)
- [路由缓存](#route-caching)

<a name="introduction"></a>
## 前言

控制器允许你讲相应的路由业务逻辑封装在控制器类中进行有效的管理，这样你不必将所有的路由逻辑集中到 `routes.php` 文件里，导致代码的臃肿与难以维护。控制器可以把关联的 HTTP 请求逻辑集中到一个类中。所有的控制器都被存储在 `app/Http/Controllers` 目录中。

<a name="basic-controllers"></a>
## 基础控制器

一个基础的控制器应该是继承自 `App\Http\Controllers\Controller` 控制器类：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

我们可以通过下面的方式把控制器的动作分配到路由：

    Route::get('user/{id}', 'UserController@showProfile');

现在，当请求匹配到指定的路由 URI 时，`UserController` 类中的 `showProfile` 方法将会被执行。当然，路由中的参数也会被直接传递到这个方法中。

#### 控制器 & 命名空间

你应该知道我们在定义控制器路由时是不需要指定完整的控制器命名空间的，而只需要指定到类名就可以了，这是因为在 `RouteServiceProvider` 文件中加载 `routes.php` 文件时，已经指定了路由组的根命名空间为 `App\Http\Controllers`。

如果你想在 `App\Http\Controllers` 目录下使用深层次 PHP 命名空间来嵌套或组织控制器，那么你只需要简单的指定相对于 `App\Http\Controllers` 部分的类名就可以了。所以如果你的控制器的全部类名为 `App\Http\Controllers\Photos\AdminController`，那么你就可以这样来定义控制器路由：

    Route::get('foo', 'Photos\AdminController@method');

#### 命名控制器路由

就像定义命名路由一样，你也可以为控制器路由指定一个名称：

    Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

你也可以使用 `route` 帮助方法来为已命名的控制器路由生成 URL:

    $url = route('name');

<a name="controller-middleware"></a>
## 控制器中间件

[中间件](/docs/{{language}}/{{version}}/middleware) 可以这样被分配到控制器路由中：

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'UserController@showProfile'
    ]);

事实上，在控制器的构造函数中指定中间件更为便捷。你可以在控制器的构造函数中使用 `middleware` 方法来轻松的分配中间件到控制器中，你甚至可以只允许类中的某些方法受到中间件的约束：

    class UserController extends Controller
    {
        /**
         * Instantiate a new UserController instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log', ['only' => [
                'fooAction',
                'barAction',
            ]]);

            $this->middleware('subscribed', ['except' => [
                'fooAction',
                'barAction',
            ]]);
        }
    }

<a name="restful-resource-controllers"></a>
## RESTful 资源控制器

资源控制器可以使你快速的构建 RESTful 类型的控制器。你可能会希望创建一个控制器来专门处理关于 "photos" 的请求。使用 `make:controller` Artisan 命令，我们可以快速的生成这种控制器：

    php artisan make:controller PhotoController --resource

这个命令会生成 `app/Http/Controllers/PhotoController.php` 文件。这个控制器中将包含每个可用的资源操作相应的方法。

接着，你可以使用下面的方式将资源路由到控制器：

    Route::resource('photo', 'PhotoController');

这一个简单的路由声明会创造多条路由用来处理 RESTful 式的请求。相应的，通过命令生成的资源型控制器也会为这些请求设置了对应的处理方法。

#### 资源控制器中的处理动作

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photo`              | index        | photo.index
GET       | `/photo/create`       | create       | photo.create
POST      | `/photo`              | store        | photo.store
GET       | `/photo/{photo}`      | show         | photo.show
GET       | `/photo/{photo}/edit` | edit         | photo.edit
PUT/PATCH | `/photo/{photo}`      | update       | photo.update
DELETE    | `/photo/{photo}`      | destroy      | photo.destroy

还记得吗，由于 HTML 表单不能支持 PUT，PATCH，或者 DELETE 类型的请求，所以你需要添加一个隐藏的 `_method` 字段来模拟相应的 HTTP 请求:

    <input type="hidden" name="_method" value="PUT">

<a name="restful-partial-resource-routes"></a>
#### 部分资源路由

当定义一个资源路由时，你也可以指定只为某些动作分配路由：

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

<a name="restful-naming-resource-routes"></a>
#### 命名资源路由

默认的，所有的资源控制器动作都被进行了相应的路由命名，你可以通过 `names` 参数来进行重命名:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
#### 命名资源路由参数

默认的，资源路由的路由参数都被命名为相应的资源名称，你可以使用 `parameters` 参数来进行重命名，`parameters` 数组应该是一个资源名称和参数名称所组成的关联数组:

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

上面的示例将会为资源的 `show` 路由生成下面的 URLs:

    /user/{admin_user}

有时候你可能希望资源路由的路由参数并不需要像默认的资源名称一样采取复数的形式，那么你可以通过传递 `parameters` 的选项为 `singular`:

    Route::resource('users.photos', 'PhotoController', [
        'parameters' => 'singular'
    ]);

    // /users/{user}/photos/{photo}

另外，你可以全局设置你的资源路由参数为单数形式或者全局进行资源路由参数的命名映射：

    Route::singularResourceParameters();

    Route::resourceParameters([
        'user' => 'person', 'photo' => 'image'
    ]);

当你自定义资源的参数时，你应该清楚的知道命名的顺序优先级：

1. 参数被直接的传递给 `Route::resource`。
2. 通过 `Route::resourceParameters` 进行全局参数映射。
3. 通过 `parameters` 数组选项传递给 `Route::resource` 或者通过 `Route::singularResoureParameters` 进行单数形式参数设置。
4. 默认的行为。

<a name="restful-supplementing-resource-controllers"></a>
#### 资源控制器中意外的行为

如果你必须在资源控制器中添加额外的行为去注册相应的路由，那么你一定要在使用 `Route::resource` 之前进行注册，否则该行为很可能会被资源控制器意外的覆盖掉。

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## 依赖注入 & 控制器

#### 构造器注入

Laravel [服务容器](/docs/{{language}}/{{version}}/container) 支持所有的 Laravel 控制器的解析。由于这个原因，你可以在控制器的构造函数中添加你所需要依赖的相应的类型提示，这些依赖会被自动的解析并注入进控制器实例中:

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

当然，你也可以对任意的 [Laravel contract](/docs/{{language}}/{{version}}/contracts) 进行类型提示，只要服务容器可以正确的解析，那么你都可以对其进行类型提示。

#### 方法中注入

出了在构造函数中注入之外，你也可以在控制器的方法中进行类型提示的依赖注入。比如，将 `Illuminate\Http\Request` 实例注入到控制器的 `store` 方法中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Store a new user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $name = $request->input('name');

            //
        }
    }

如果你的控制器方法中也接收来自路由传递的参数，那么它们会在其它依赖解析完毕之后被传递，比如你的路由是这么定义的：

    Route::put('user/{id}', 'UserController@update');

你仍然可以在控制器方法中添加 `Illuminate\Http\Request` 的类型提示，然后添加路由的参数 `id` 就像下面这样：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * Update the specified user.
         *
         * @param  Request  $request
         * @param  string  $id
         * @return Response
         */
        public function update(Request $request, $id)
        {
            //
        }
    }

<a name="route-caching"></a>
## 路由缓存

> **注意** 路由缓存不支持基于闭包函数定义的路由。如果你想使你的路由被缓存，那么你应该使用控制器来管理你的路由。

如果你所有的路由都是基于控制器的路由，那么你应该使用 Laravel 所推荐的路由缓存，使用路由缓存，可以大大的减少路由注册到应用中所需要的时间，从而提高应用的响应速度。在一些极端的案例中，你的路由注册甚至能提升 100 倍的速度。你可以通过使用 `route:cache` Artisan 命令来生成路由缓存:

    php artisan route:cache

这就是了！你缓存的路由文件现在将会取代你的 `app/Http/routes.php` 文件，请记住，如果你新增了任意路由，你都需要重新生成缓存，正是出于这个原因，你应该只在项目的部署环节使用 `route:cache` 命令。

最后，你可以使用 `route:clear` 命令来清除路由缓存：

    php artisan route:clear
