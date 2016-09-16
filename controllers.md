# 控制器

- [前言](#introduction)
- [基础控制器](#basic-controllers)
    - [定义控制器](#defining-controllers)
    - [控制器 & 命名空间](#controllers-and-namespaces)
    - [单一行为的控制器](#single-action-controllers)
- [控制器中间件](#controller-middleware)
- [资源控制器](#resource-controllers)
    - [部分资源路由](#restful-partial-resource-routes)
    - [命名资源路由](#restful-naming-resource-routes)
    - [命名资源路由参数](#restful-naming-resource-route-parameters)
    - [意外的行为](#restful-supplementing-resource-controllers)
- [依赖注入 & 控制器](#dependency-injection-and-controllers)
- [路由缓存](#route-caching)

<a name="introduction"></a>
## 前言

控制器允许你将相应的路由业务逻辑封装在控制器类中进行有效的管理，这样你不必将所有的路由逻辑集中到路由文件里，导致代码的臃肿与难以维护。控制器可以把关联的 HTTP 请求逻辑集中到一个类中。所有的控制器都被存储在 `app/Http/Controllers` 目录中。

<a name="basic-controllers"></a>
## 基础控制器

<a name="defining-controllers"></a>
### 定义控制器

下面的示例是一个基本的控制器。你需要注意的是这个控制器继承了 Laravel 所提供的基础控制器。基础控制器提供了一些方便的方法，比如 `middleware` 方法可以用来为控制器的动作附加中间件:

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
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

你可以通过下面的方式把控制器的动作分配到路由：

    Route::get('user/{id}', 'UserController@show');

现在，当请求匹配到指定的路由 URI 时，`UserController` 类中的 `show` 方法将会被执行。当然，路由中的参数也会被直接传递到这个方法中。

> {tip} 控制器并不是 **必须** 继承自基础控制器的。但是不继承的话，你就不能访问如 `middleware`，`validate`，和 `dispatch` 这些方便的方法了。

<a name="controllers-and-namespaces"></a>
### 控制器 & 命名空间

你应该知道我们在定义控制器路由时是不需要指定完整的控制器命名空间的，而只需要指定到类名就可以了，这是因为在 `RouteServiceProvider` 文件加载你的路由文件时，已经指定了路由组的根命名空间为 `App\Http\Controllers`，所以我们只需要输入相对于命名空间 `App\Http\Controllers` 之后的类名就可以了。

如果你想在 `App\Http\Controllers` 目录下使用深层次的目录来嵌套或组织控制器，那么你只需要简单的指定相对于 `App\Http\Controllers` 部分的类名就可以了。所以如果你的控制器的全部类名为 `App\Http\Controllers\Photos\AdminController`，那么你就可以这样来定义控制器路由：

    Route::get('foo', 'Photos\AdminController@method');

<a name="single-action-controllers"></a>
### 单一行为的控制器

如果你需要定义一个控制器却只处理一种行为，那么你可以只在控制器中放置一个单一的 `__invoke` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Http\Controllers\Controller;

    class ShowProfile extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function __invoke($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

当为单一行为的控制器定义路由时，你并不需要指定任何方法：

    Route::get('user/{id}', 'ShowProfile');

<a name="controller-middleware"></a>
## 控制器中间件

[中间件](/{{language}}/{{version}}/middleware) 可以这样被分配到控制器路由中：

    Route::get('profile', 'UserController@show')->middleware('auth');

事实上，在控制器的构造函数中指定中间件更为便捷。你可以在控制器的构造函数中使用 `middleware` 方法来轻松的分配中间件到控制器的动作中，你甚至可以只允许类中的某些方法受到中间件的约束：

    class UserController extends Controller
    {
        /**
         * Instantiate a new new controller instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log')->only('index');

            $this->middleware('subscribed')->except('store');
        }
    }

> {tip} 你可以分配中间件到控制器的一部分动作中。但不管怎样，它或许可以表明你的控制器在成长为一个庞大的控制器。所以，或许你可以考虑将这些大型的控制器分散为更小的多个控制器。

<a name="resource-controllers"></a>
## 资源控制器

Laravel 的资源路由可以在一行代码中分配普通的 "CRUD" 路由到控制器中。比如，你可能会希望创建一个控制器来专门处理关于 "photos" 的请求。使用 `make:controller` Artisan 命令，我们可以快速的生成这种控制器：

    php artisan make:controller PhotoController --resource

这个命令会生成 `app/Http/Controllers/PhotoController.php` 文件。这个控制器中将为每个可用的资源操作分配相应的方法。

接着，你可以使用下面的方式将资源路由到控制器：

    Route::resource('photos', 'PhotoController');

这一个简单的路由声明会创造多条路由用来处理多种资源相关动作的请求。相应的，通过命令生成的资源型控制器也会为这些请求设置了对应的处理方法。

#### 资源控制器中的处理动作

Verb      | URI                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### 虚假的表单方法

由于 HTML 表单不能支持 `PUT`，`PATCH`，或者 `DELETE` 类型的请求，所以你需要添加一个隐藏的 `_method` 字段来模拟相应的 HTTP 请求，你可以使用 `method_field` 帮助方法来进行创建:

    {{ method_field('PUT') }}

<a name="restful-partial-resource-routes"></a>
### 部分资源路由

当定义一个资源路由时，你也可以指定只为某些动作分配路由，而不必为所有动作分配相应的路由：

    Route::resource('photo', 'PhotoController', ['only' => [
        'index', 'show'
    ]]);

    Route::resource('photo', 'PhotoController', ['except' => [
        'create', 'store', 'update', 'destroy'
    ]]);

<a name="restful-naming-resource-routes"></a>
### 命名资源路由

默认的，所有的资源控制器动作都有一个相应的路由命名，你可以通过可选的 `names` 数组参数来进行重命名:

    Route::resource('photo', 'PhotoController', ['names' => [
        'create' => 'photo.build'
    ]]);

<a name="restful-naming-resource-route-parameters"></a>
### 命名资源路由参数

默认的，资源路由的路由参数都被命名为相应的资源名称的单数形式，你可以使用 `parameters` 参数来进行重命名，`parameters` 数组应该是一个资源名称和参数名称所组成的关联数组:

    Route::resource('user', 'AdminUserController', ['parameters' => [
        'user' => 'admin_user'
    ]]);

上面的示例将会为资源的 `show` 路由生成下面的 URLs:

    /user/{admin_user}

<a name="restful-supplementing-resource-controllers"></a>
#### 意外的行为

如果你必须在资源控制器中添加额外的行为去注册相应的路由，那么你一定要在使用 `Route::resource` 之前进行注册，否则该行为很可能会被资源控制器意外的覆盖掉。

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} 请保持对你的控制器的聚焦。如果你发现了你的资源控制器动作中包含了一些通用方法，你可以考虑将你的控制器分离到两个比较小的控制器中。

<a name="dependency-injection-and-controllers"></a>
## 依赖注入 & 控制器

#### 构造器注入

Laravel [服务容器](/{{language}}/{{version}}/container) 支持所有的 Laravel 控制器的解析。由于这个原因，你可以在控制器的构造函数中添加你所需要依赖的相应的类型提示，这些依赖会被自动的解析并注入进控制器实例中:

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

当然，你也可以对任意的 [Laravel contract](/{{language}}/{{version}}/contracts) 进行类型提示，只要服务容器可以正确的解析，那么你都可以对其进行类型提示。在你的控制器中提供依赖注入，可以其具有更高的可测试性。

#### 方法注入

除了在构造函数中注入之外，你也可以在控制器的方法中进行类型提示的依赖注入。比如，将 `Illuminate\Http\Request` 实例注入到控制器的 `store` 方法中：

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
            $name = $request->name;

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
         * Update the given user.
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

> {note} 路由缓存不支持基于闭包函数定义的路由。如果你想使你的路由被缓存，那么你应该使用控制器来管理你的路由。

如果你所有的路由都是基于控制器的路由，那么你应该使用 Laravel 所推荐的路由缓存，使用路由缓存，可以大大的减少路由注册到应用中所需要的时间，从而提高应用的响应速度。在一些极端的案例中，你的路由注册甚至能提升 100 倍的速度。你可以通过使用 `route:cache` Artisan 命令来生成路由缓存:

    php artisan route:cache

在执行这条命令之后，你缓存的路由文件现在将会取代你的定义路由文件，所有的请求都会自动加载缓存的路由文件，请记住，如果你新增了任意路由，你都需要重新生成缓存，正是出于这个原因，你应该只在项目的部署环节使用 `route:cache` 命令。

最后，你可以使用 `route:clear` 命令来清除路由缓存：

    php artisan route:clear
