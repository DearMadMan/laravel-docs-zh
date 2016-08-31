# HTTP Session

- [前言](#introduction)
    - [配置](#configuration)
    - [驱动依赖](#driver-prerequisites)
- [使用 Session](#using-the-session)
    - [检索数据](#retrieving-data)
    - [存储数据](#storing-data)
    - [闪存数据](#flash-data)
    - [删除数据](#deleting-data)
    - [重置 Session ID](#regenerating-the-session-id)
- [添加自定义 Session 驱动](#adding-custom-session-drivers)
    - [实现驱动](#implementing-the-driver)
    - [注册驱动](#registering-the-driver)

<a name="introduction"></a>
## 前言

由于 HTTP 驱动是一种无状态的协议，这通常意味着服务端并不能清楚的知道当前请求用户与之前请求用户间的关系，而 Session 提供了这种跨请求用户的解决方案。Laravel 附带了简洁统一的 API 来支持各种后端 session 驱动。对于 [Memcached](http://memcached.org/)，[Redis](http://redis.io/) 和数据库这种流行的后端驱动，Laravel 提供了开箱即用的支持。

<a name="configuration"></a>
### 配置

session 的配置文件被存储在 `config/session.php`。你应该确保在使用前阅读该文件中的配置项注释。Laravel 对配置中的每一项都进行了详细的注释。默认的，Laravel 配置使用的是 `file` session 驱动，这对于大多数应用来说已经够用了。但是在生产环境中还是建议使用 `memcached` 或者 `redis` 这种内存级驱动，这对于应用的 session 性能来说会是一个很大的提升。

对于每个请求的 session 数据，都会存储在你所定义的 session `driver` 所在的位置中。下列驱动在 Laravel 中可以开箱即用：

<div class="content-list" markdown="1">
- `file` - sessions 会存储在 `storage/framework/sessions` 中。
- `cookie` - sessions 会被存储在经过安全加密的 cookies 中。
- `database` - sessions 会被存储在你应用中所使用的数据库中。
- `memcached` / `redis` - sessions 会被存储在更快的内存级存储驱动中。
-  `array` - sessions 会使简单存储在 PHP 的数组中，并且它不能被跨请求访问。
</div>

> {tip}：`array` 驱动通常是用来进行 [测试](/docs/{{language}}/{{version}}/testing) 时使用的以避免持久的 session 数据。

<a name="driver-prerequisites"></a>
### 驱动依赖

#### 数据库

当使用 `database` session 驱动时，你需要先创建一个表来存储 session 项。下面的例子是数据库 session 驱动表的架构声明：

    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->integer('user_id')->nullable();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity');
    });

你也可以使用 `session:table` Artisan 命令来生成一个数据库会话驱动的迁移表：

    php artisan session:table

    php artisan migrate

#### Redis

在使用 Redis session 驱动之前，你需要通过 Composer 安装 `predis/predis`（~1.0）。你可以在 `database` 配置文件中对你的 Redis 连接进行配置。在 `session` 配置文件中，`database` 选项可以用来指定 session 应该使用哪个 Redis 连接。

<a name="using-the-session"></a>
## 使用 Session

<a name="retrieving-data"></a>
### 检索数据

在 Laravel 中有两种主要的方式来与 session 进行交互：`session` 帮助函数和 `Request` 实例。首先，让我们通过 `Request` 实例来访问 session，我们可以可以将请求类在控制器方法中进行类型提示。还记得吗，控制器方法中的依赖会通过 Laravel 的 [服务容器](/docs/{{language}}/{{version}}/container) 自动的注入:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function show(Request $request, $id)
        {
            $value = $request->session()->get('key');

            //
        }
    }

当你从 session 中检索值时，你也可以指定一个默认的值到 `get` 方法的第二个参数，默认值会在 session 未检索到相应键的值时返回。你也可以传递 `Closure` 作为默认值，如果找不到相应的键，`Closure` 执行的结果所返回的值将作为默认值：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function() {
        return 'default';
    });

#### 全局 Session 帮助方法

你也可以使用全局 `session` 方法来检索和存储数据到 session 中。当 `session` 方法调用并伴随一个单个字符串作为参数时，它会返回 session 相应键中的值。当 `session` 方法被调用并伴随一个键值对所组成的数组作为参数时，那么这些值将会存储到 session 中:

    Route::get('home', function () {
        // Retrieve a piece of data from the session...
        $value = session('key');

        // Specifying a default value...
        $value = session('key', 'default');

        // Store a piece of data in the session...
        session(['key' => 'value']);
    });

> {tip} 在实际使用时不论是使用 HTTP 请求实例还是通过全局帮助方法 `session` 都没有太多的区别。两个方法都是可以在测试用例中通过 `assertSessionHas` 方法进行 [测试的](/docs/{{language}}/{{version}}/testing)。

#### 检索所有 Session 数据

如果你希望从 session 中检索出所有的值，你可以使用 `all` 方法：

    $data = $request->session()->all();

#### 判断 Session 中项是否存在

你可以使用 `has` 方法来判断 session 中是否含有某项，如果存在则返回 `true`，如果不存在则返回 `null`:

    if ($request->session()->has('users')) {
        //
    }

如果是为了判断 session 中是否存在某项的值，并且即使它的值是 `null`，那么你可以使用 `exists` 方法，该方法会在 session 存在某键时返回 `true`:

    if ($request->session()->exists('users')) {
        //
    }

<a name="storing-data"></a>
### 存储数据

你通常可以使用 `put` 方法或者 `session` 帮助方法来将数据存储到 session 中：

    // Via a request instance...
    $request->session()->put('key', 'value');

    // Via the global helper...
    session(['key' => 'value']);

#### 推送值到 session 的数组项中

你可以使用 `push` 方法来将一个新的值推送到 session 中值为数组的项中。比如，如果 `user.teams` 键表明了数组在 session 中的路径，你可以像这样推送新的值到该数组中：

    $request->session()->push('user.teams', 'developers');

#### 检索 & 删除项

`pull` 方法会在检索的同时在 session 中删除该项：

    $value = $request->session()->pull('key', 'default');

<a name="flash-data"></a>
### 闪存数据

有时候你希望在 session 中存储某些数据，但是只允许下一次请求时使用，使用后即消除。那么就可以使用 `flash` 方法。这方法在 session 中存储的数据只能在随后的请求中进行访问，并且在之后删除。闪存的数据通常用来做一些临时的状态消息：

    $request->session()->flash('status', 'Task was successful!');

如果你需要维持闪存的数据到更多的请求，你可以使用 `reflash` 方法，它会维持所有的闪存数据到额外的请求中。如果你只需要维持指定的闪存数据，你可以使用 `keep` 方法：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

<a name="deleting-data"></a>
### 删除数据

你可以使用 `forget` 方法来删除 session 中的某一项数据，如果你想要移除 session 中的所有数据，你可以使用 `flush` 方法：

    $request->session()->forget('key');

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### 重新生成 Session ID

重新生成 session ID 通常是为了避免恶意用户在你的应用中利用 [session 定位](https://en.wikipedia.org/wiki/Session_fixation) 进行攻击。

如果你是使用內建的 `LoginController` 进行用户认证的，那么 Laravel 会自动的生成 session ID。然而，如果你需要手动的重置 session ID，那么你可以使用 `regenerate` 方法：

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## 添加自定义 Session 驱动

<a name="implementing-the-driver"></a>
#### 实现驱动

你应该让你的自定义 session 驱动实现 `SessionHandlerInterface` 接口。该接口仅包含了几个简单的需要实现的方法。一个实现了的 MongoDB 驱动的结构应该像下面一样：

    <?php

    namespace App\Extensions;

    class MongoHandler implements SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> {tip} Laravel 本身并没有为你的扩展构建一个默认的目录。你可以按照自己的喜欢将其放置在任意的位置。比如，你可以创建一个 `Extensions` 目录来放置 `MongoHandler` 扩展。

由于这些方法的意图并不是那么易于理解的，所以让我们来快速的来了解一下各种方法都做了些什么吧：

<div class="content-list" markdown="1">
- `open` 方法一般用于基于文件的 session 存储系统。由于 Laravel 已经附带了 `file` session 驱动，所以你几乎不需要在这个方法中添加任何的代码。你可以直接放置其为空方法。事实上，这是一种可怜的接口设计（我们将会在后面讨论它），但是 PHP 却必须要求我们实现这个方法。
- `close` 方法，类似于 `open` 方法，你也可以对其进行忽视，对于大多数驱动来说根本不需要。
- `read` 方法应该根据给定的 `$sessionId` 返回 session 中关联数据的字符串版本。你不需要在存储或检索时做任何的序列化或其它的编码操作，因为 Laravel 会为你执行序列化服务。
- `write` 方法应该将给定的 `$data` 字符串关联到 `$sessionId` 并写入持续的存储系统中，比如 MongoDB，Dynamo，等。还有，你不应该提供任何的序列化服务，因为 Laravel 会为你处理这些。
- `destroy` 方法应该从 session 存储中删除所关联的数据。
- `gc` 方法应该根据给定的 `$lifetime` UNIX 时间戳删除 session 所关联的过期数据。对于拥有自动过期能力的系统，比如 Memcached 和 Redis，你可以直接让这个方法为空。
</div>

<a name="registering-the-driver"></a>
#### 注册驱动

当你完成驱动的编写之后，你就可将之注册到框架中了。你需要使用 `Session` [假面](/docs/{{language}}/{{version}}/facades) 的 `extend` 方法才能添加额外的 session 驱动到 Laravel 中。你应该在 [服务提供者](/docs/{{language}}/{{version}}/providers) 的 `boot` 方法中来调用 `extend` 方法：

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionStore;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Session::extend('mongo', function($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionStore;
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

一旦你的 session 驱动被注册。你就可以在 `config/session.php` 配置文件中使用 `mongo` 驱动了。
