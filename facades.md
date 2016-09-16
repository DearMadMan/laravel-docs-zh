# 假面

- [前言](#introduction)
- [何时使用](#when-to-use-facades)
    - [假面 Vs. 依赖 注入](#facades-vs-dependency-injection)
    - [假面 Vs. 帮助方法](#facades-vs-helper-functions)
- [假面工作原理](#how-facades-work)
- [假面类参考](#facade-class-reference)

<a name="introduction"></a>
## 前言

假面提供了一种类似‘静态’接口的方式来从 [服务容器](/{{language}}/{{version}}/container) 中提取可用类的方法。Laravel 自身附带了许多的假面，你可能在还不知道它的时候就已经使用过它了。Laravel 的假面服务就好像是一个从服务容器中取出底层类的代理（你带上我的面具，你就可以使用我的方法了），它提供了一种简洁语法的同时保持了比传统静态方法更高的可测试性和灵活性。

Laravel 中所有的假面都被定义在 `Illuminate\Support\Facades` 命名空间下，所以，我们可以像这样来访问假面：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

贯穿整个 Laravel 文档，很多示例都是使用假面来演示框架中的各种特性。

<a name="when-to-use-facades"></a>
## 何时使用

假面拥有多种益处。它们提供简介醒目的语法来使你可以快速的使用 Laravel 的特性，而不需要强制的进行依赖注入或者手动的写出一大堆的类名。幸运的是，正是由于它们独特的使用 PHP 的动态方法，这导致它们更易于测试。

但是不管怎么样，你在使用假面时必须要注意一些事情。其最大的危害就是假面类的滥用。由于假面使用起来非常简单，它并不需要注入依赖，所以它可能会使你不知不觉中在一个类文件中包含了大量的假面。而使用依赖注入，这种潜在的危害会在你的构造函数变的庞大时就有非常显著的视觉反馈。所以，当使用假面时，你应该保持注意类的大小，你应该限制类的职责在一个狭窄的范围呢。

> {tip} 当构建第三方扩展包与 Laravel 进行交互时，最好是注入 [Laravel 契约](/{{language}}/{{version}}/contracts) 的方式来取代使用假面。这是由于扩展包是在 Laravel 之外进行构建的，你并不能在测试时得到 Laravel 对于假面测试的帮助。

<a name="facades-vs-dependency-injection"></a>
### 假面 Vs. 依赖注入

依赖注入的最大的益处就是其可以无痛的切换所注入类的实现。这通常有益于在测试时注入模拟或桩对象，并通过桩对象来断言各种方法是否被调用。

通常，这是不可能去模拟或者桩出一个真实的静态类方法的。但是，由于假面是使用的动态方法来代理从服务容器中解析获得的对象的方法，所以我们其实可以就像测试一个注入的实例那样来测试假面。举个例子，给定以下路由：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

我们可以编写一个下面测试用例来验证 `Cache::get` 方法是否被正确的调用，并传递了所期望的参数：

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="facades-vs-helper-functions"></a>
### 假面 Vs. 帮助方法

除了假面之外，Laravel 也包含了各种帮助方法，这些帮助方法可以提供一些常用的任务，比如生成视图，触发事件，发布任务，或者发送 HTTP 响应。对于多数这类帮助方法，Laravel 也同样提供了相应的假面服务。比如，下面假面的调用和帮助方法的调用时等量的：

    return View::make('profile');

    return view('profile');

实际上使用假面和使用帮助方法是没有什么区别的。当你使用帮助方法时，你仍然需要使用相应的假面来进行测试。比如，给定下面的路由：

    Route::get('/cache', function () {
        return cache('key');
    });

事实上，`cache` 帮助方法也是借助底层 `Cache` 假面的 `get` 方法的调用。所以，即使我们是使用的帮助方法，我们也要编写类似下面的测试用例来验证相应的方法是否被调用并传递了正确的参数：

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="how-facades-work"></a>
## Facades 工作原理

在 Laravel 中，假面是一种可以从容器中访问相应对象的类。这种机制让其可以在 `Facade` 类中进行工作。Laravel 的假面和任何自定义的假面类都需要继承自基础的 `Illuminate\Support\Facades\Facade` 类。

基础的 `Facade` 类使用 `__callStatic()` 魔术方法来从服务容器中解析相应的对象中调用相应的方法。在下面的示例中，Laravel 缓存系统的方法将会被调用，如果你只是匆匆的看一眼，你可能会觉得它调用的是 `Cache` 类的 `get` 静态方法：

    <?php

    namespace App\Http\Controllers;

    use Cache;
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
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

你应该注意到我们在文件的顶部引入的是 `Cache` 假面。这个假面服务相当于访问 `Illuminate\Contracts\Cache\Factory` 接口底层实现的代理。所有使用假面调用的方法都会在 laravel 缓存服务的底层实现进行传递调用。

如果我们看一下 `Illuminate\Support\Facades\Cache` 类，你会发现这里并没有静态方法 `get`:

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

事实上，`Cache` 假面继承自基类 `Facade` 并且定义了 `getFacadeAccessor()` 方法。这个方法的主要任务就是从服务容器中返回服务的绑定名称。当用户调用任何 `Cache` 假面的静态方法时，Laravel 从 [服务容器](/{{language}}/{{version}}/container) 中返回绑定了 `cache` 为名称的对象，并且使用这个对象调用所请求的方法。

<a name="facade-class-reference"></a>
## 假面类参考

在下面的表格中你会找到所有的假面及其所对应的底层实现类。这是一个可以迅速挖掘给定假面 API 的有用的工具。在 [服务容器](/{{language}}/{{version}}/container) 中所给的绑定键也包括在其中。

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](http://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](http://laravel.com/api/5.1/Illuminate/Contracts/Auth/Access/Gate.html)  |
Hash  |  [Illuminate\Contracts\Hashing\Hasher](http://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/{{version}}/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue`
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/{{version}}/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/{{version}}/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |
Storage  |  [Illuminate\Contracts\Filesystem\Factory](http://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Factory.html)  |  `filesystem`
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/{{version}}/Illuminate/View/View.html)  |
