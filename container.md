# 服务容器

- [前言](#introduction)
- [绑定](#binding)
    - [绑定基础](#binding-basics)
    - [绑定接口到实现](#binding-interfaces-to-implementations)
    - [上下文绑定](#contextual-binding)
    - [标签](#tagging)
- [解析](#resolving)
    - [Make 方法](#the-make-method)
    - [自动注入](#automatic-injection)
- [容器事件](#container-events)

<a name="introduction"></a>
## 前言

Laravel 提供了强大的服务容器工具来管理类之间的依赖关系并以此来提供依赖注入。依赖注入是一个奇特的短语，其实它的意思是说所依赖的类的实例通过构造函数或者其它的方式被注入到类中。

> {tip} 如果你对依赖注入有太多的疑惑，你可以参考我写的这篇 [《理解依赖注入与控制反转》](http://team.dearmadman.com/2016/04/13/dependency-injection-and-inversion-of-control/)。

让我们来看一个简单的例子:

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use App\Repositories\UserRepository;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
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

        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }
在上面的例子中，`UserController` 需要从数据源中获取到用户的数据，所以我们将 `注入` 一个服务，这个服务可以检索出用户。在这种场景下，我们的 `UserRepository` 更像是使用 [Eloquent](/docs/{{language}}/{{version}}/eloquent) 来从数据库中检索出用户信息。但是不管怎么样，由于存储库的被注入，我们可以非常轻松的换成其它的实现。我们也可以非常轻松的去模拟数据，或者是在进行测试时创建一个假的 `UserRepository` 来使用。


能够更深层次的理解 laravel 的服务容器是非常必要的，它是能够构造强大的应用的关键，这对于贡献 Laravel 内核是非常有帮助的。

<a name="binding"></a>
## 绑定

<a name="binding-basics"></a>
### 绑定基础


几乎所有的服务容器的绑定都是在 [服务提供者](/docs/{{language}}/{{version}}/providers) 中被注册的，所有下面的例子都是在这个场景下进行演示的。

> {tip} 如果类并没有依赖什么接口，它是没有必要在容器中进行绑定的。容器并不需要有什么具体的指示去如何构造这些实例，因为他们会根据 PHP 的反射进行自动的实例化。

#### 简单绑定

在服务提供者中，你总是可以通过使用 `$this->app` 属性来访问底层的容器。我们可以使用 `bind` 方法来注册绑定，你可以传递类名或者接口的名字，然后接着一个 `Closure` 函数返回相关的实例。

    $this->app->bind('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

你需要注意的是，我们将容器的实例作为参数传递到闭包中，这样我们就可以使用容器来解析我们所构造对象的子依赖。

#### 绑定一个单例

`singleton` 方法绑定的类或者接口只会被解析一次，也就是说随后的访问都会返回这个相同的实例：

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new HelpSpot\API($app->make('HttpClient'));
    });

#### 绑定实例

你也可以使用 `instance` 方法来绑定一个已经存在的实例对象到容器中。随后的访问中，容器都会返回这个给定的实例：

    $api = new HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\Api', $api);

#### 绑定原始类型

有时候你可能需要在类中注入许多的依赖类，但是你可能也需要注入一些 PHP 的原始类型数据，比如说 integer, boolean。你可以在上下文绑定中非常轻松的绑定类所需要的：

    $this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);

<a name="binding-interfaces-to-implementations"></a>
### 绑定接口到实现

服务容器具有一个非常强大的特性就是能够绑定接口到给定的实现。比如，让我们假设我们有 `EventPusher` 接口和 `RedisEventPusher` 实现。一旦我们绑定 `RedisEventPusher` 实现到这个接口，我们可以在服务容器中这么来进行注册：

    $this->app->bind(
        'App\Contracts\EventPusher',
        'App\Services\RedisEventPusher'
    );

这就告诉了容器如果类需要一个 `EventPusher` 的实现时，那就注入一个 `RedisEventPusher` 的实例。现在我们可以在构造函数或者其他地方来写入 `EventPusher` 接口的类型提示来进行依赖注入：

    use App\Contracts\EventPusher;

    /**
     * Create a new class instance.
     *
     * @param  EventPusher  $pusher
     * @return void
     */
    public function __construct(EventPusher $pusher)
    {
        $this->pusher = $pusher;
    }

<a name="contextual-binding"></a>
### 上下文绑定

有时候你可能会有实现了同一个接口的两个类，而且你想要在不同的场景下注入不同的实现，比如说，两个控制器可以各自依赖 `Illuminate\Contracts\Filesystem\Filesystem` [contract](/docs/{{language}}/{{version}}/contracts) 的不同实现。Laravel 提供了一种简单流利的接口来定义这种行为：

    use Illuminate\Support\Facades\Storage;
    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\VideoController;
    use Illuminate\Contracts\Filesystem\Filesystem;

    $this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });

    $this->app->when(VideoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('s3');
              });

<a name="tagging"></a>
### 标记

有时候你可能需要解析一些符合某种类别的所有绑定。比如说，你可能需要来建造一个报告聚合器来接收一个包含了不同 `Report` 接口的实现集。在实现了 `Report` 的注册之后，你可以使用 `tag` 方法来给它们打上标记：

    $this->app->bind('SpeedReport', function () {
        //
    });

    $this->app->bind('MemoryReport', function () {
        //
    });

    $this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

一旦这些服务被打上标记，你可以使用 `tagged` 方法来解析获得它们:

    $this->app->bind('ReportAggregator', function ($app) {
        return new ReportAggregator($app->tagged('reports'));
    });

<a name="resolving"></a>
## 解析

<a name="the-make-method"></a>
#### `make` 方法

你可以使用 `make` 方法从容器中解析类的实例，它接收一个你想要解析的类或者接口的名称：

    $api = $this->app->make('HelpSpot\API');

如果你所处的位置并不能访问 `$app` 变量，那么你可以使用全局帮助函数 `resolve` 方法：

    $api = resolve('HelpSpot\API');

<a name="automatic-injection"></a>
#### 自动注入

另外，也是最重要的，你也可以在类的构造函数中简单的使用‘类型提示’来从容器中解析依赖。[控制器](/docs/{{language}}/{{version}}/controllers)，[事件监听器](/docs/{{language}}/{{version}}/events)，[队列任务](/docs/{{language}}/{{version}}/queues)，[中间件](/docs/{{language}}/{{version}}/middleware)等都支持这种方式。在实践中，这是大部分对象从容器中解析的方式。

比如，你可以在控制器的构造函数中对应用中定义的存储库类进行类型提示。存储库实例会自动的从容器中进行解析并注入到控制器中:

    <?php

    namespace App\Http\Controllers;

    use App\Users\Repository as UserRepository;

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

        /**
         * Show the user with the given ID.
         *
         * @param  int  $id
         * @return Response
         */
        public function show($id)
        {
            //
        }
    }

<a name="container-events"></a>
## 容器事件

服务容器在每次解析一个对象时都会触发一个事件，你可以通过 `resolving` 方法来监听这个事件:

    $this->app->resolving(function ($object, $app) {
        // Called when container resolves object of any type...
    });

    $this->app->resolving(HelpSpot\API::class, function ($api, $app) {
        // Called when container resolves objects of type "HelpSpot\API"...
    });
就如你所看到的，`resolving` 方法中你可以传递一个回调函数，回调函数会接收两个参数，一个是解析得到的对象，一个是容器本身，这样你就可以在对象传递到消费者之前添加一些额外的属性。
