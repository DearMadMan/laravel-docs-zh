# HTTP Middleware

- [前言](#introduction)
- [定义中间件](#defining-middleware)
- [注册中间件](#registering-middleware)
    - [全局中间件](#global-middleware)
    - [分配中间件到路由](#assigning-middleware-to-routes)
    - [中间件组](#middleware-groups)
- [中间件参数](#middleware-parameters)
- [末端中间件](#terminable-middleware)

<a name="introduction"></a>
## 前言

HTTP 中间件为你的应用提供了一种便利的机制去过滤客户端的请求，比如说 Laravel 中自带的用来验证用户是否已经认证的中间件，如果用户的认证没有通过，那么他将被重定向到登录视图。而如果用户已经通过认证，那么他的请求就会被认证中间件通过，并将请求传递给应用。

中间件可以处理多种任务，不仅仅局限于用户认证。比如你可以创建一个跨同源策略的中间件，用来处理每个请求在被响应前添加正确的响应头，你还可以创造一个日志中间件，在应用被请求时优先记录下请求信息。

Laravel 框架本身附带了一些中间件，它们包括维护、认证、CSRF 保护，session 中间件等。这些中间件都被定义在 `app\Http\Middleware` 目录中。

<a name="defining-middleware"></a>
## 定义中间件

你可以直接使用 Laravel 提供的 `make:middleware` artisan 命令来创建一个新的中间件:

    php artisan make:middleware AgeMiddleware

这条命令会在 `app\Http\Middleware` 目录下创建一个 `AgeMiddleware.php` 类文件。在这个中间件中，我们设置仅允许 age 大于 200 的请求通过，否则将其重定向到 "home" URI。

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AgeMiddleware
    {
        /**
         * Run the request filter.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->input('age') <= 200) {
                return redirect('home');
            }

            return $next($request);
        }

    }

你可以看到，如果请求中给定的 `age` 小于或等于 `200`，那么中间件将会返回一个 HTTP 重定向到客户端。其它情况下，请求将会被继续传递到应用中。为了将中间件继续将请求传递给下层应用，你需要调用 `$next` 回调函数，并将 `$request` 传递进去。

你可以把中间件想象为一系列的过滤层，HTTP 请求必须要在触碰到应用之前经过这些层，每一层都需要对请求进行检查，甚至是拒绝请求通往下一层。

### *前置* / *后置* 中间件

一个中间件是在请求触碰到应用前执行还是请求触碰到应用之后执行是由中间件自身所决定的。比如，下面的中间件就会在请求被应用处理 **前** 优先处理一些其他的任务：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddleware
    {
        public function handle($request, Closure $next)
        {
            // Perform action

            return $next($request);
        }
    }

事实上，下面的中间件会在请求被应用响应 **后** 再执行一些其他的任务:

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handle($request, Closure $next)
        {
            $response = $next($request);

            // Perform action

            return $response;
        }
    }

<a name="registering-middleware"></a>
## 注册中间件

<a name="global-middleware"></a>
### 全局中间件

如果你需要一个可以过滤所有请求的中间件，那么你可以注册一个全局中间件。你需要先定义好中间件，然后在 `app/Http/Kernel.php` 中的 `$middleware` 数组变量中进行追加注册。

<a name="assigning-middleware-to-routes"></a>
### 分配中间件到路由

如果你想要分配中间件到特定的路由，那么你需要在 `app/Http/Kernel.php` 文件中的 `$routeMiddleware` 属性中进行注册，在这里你应该定义一个短字符的别名，以便于你在路由分配时快速的指定。

    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    ];

当你在 `kernel` 文件中定义完成中间件之后，你就可以在定义路由时使用 `middleware` 选项进行中间件分配：

    Route::get('admin/profile', ['middleware' => 'auth', function () {
        //
    }]);

你可以通过数组的形式分配多个中间件到路由：

    Route::get('/', ['middleware' => ['first', 'second'], function () {
        //
    }]);

除了使用数组的方式，Laravel 也允许你通过链式的调用 `middleware` 方法在定义路由时进行路由中间件的分配：

    Route::get('/', function () {
        //
    })->middleware(['first', 'second']);

事实上，你也可以使用完全类名来进行中间件的分配：

    use App\Http\Middleware\FooMiddleware;

    Route::get('admin/profile', ['middleware' => FooMiddleware::class, function () {
        //
    }]);

<a name="middleware-groups"></a>
### 中间件组

有时候你可能希望通过一个别名来分配一系列的中间件到路由。你可以在 `Kernel` 中的 `$middlewareGroups` 为一系列路由分为一组并进行命名。

Laravel 附带了 `web` 和 `api` 两个中间件组，这两个中间件组分别包含了一系列常用于 web UI 和 API 路由的中间件:

    /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
        ],

        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

中间件组可以使用和普通中间件相同的语法分配到路由和控制器动作中。这次，中间件组非常简单方便的一次性将多个中间件分配到路由中：

    Route::group(['middleware' => ['web']], function () {
        //
    });

你需要注意的是，Laravel 自带的 `web` 中间件已经被默认启用，所有在 `routes.php` 中的被定义的路由都被分配了此中间件。你可以在 `routeServiceProvider.php` 文件中进行修改。

<a name="middleware-parameters"></a>
## 带参数的中间件

中间节也可以接受一些额外的自定义参数。比如说你可能需要在执行给定的动作之前先验证用户是否拥有所给定的权限，你可以构建一个 `RoleMiddleware` 来接受一个角色的名称作为额外的参数。

额外的中间件参数将会被传递在 `$next` 参数之后：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class RoleMiddleware
    {
        /**
         * Run the request filter.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }

            return $next($request);
        }

    }

带参数的中间件在分配给路由时需要在中间件别名之后跟 `:` 字符来分割别名和参数。对于多个参数应该使用 `,` 进行分割:

    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

<a name="terminable-middleware"></a>
## 末端中间件

有时候你可能需要在响应已经被发送到客户端之后继续处理一些任务，比如 `session` 中间件在 Laravel 中就是响应被发送出去 **之后** 才将 session 信息进行本地存储操作。你可以通过在中间件中添加 `terminate` 方法来定义一个末端中间件：

    <?php

    namespace Illuminate\Session\Middleware;

    use Closure;

    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

`terminate` 方法应该同时接收请求和响应做为参数。当你完成末端中间件的构建时，你应该在 `Kernel` 中将它注册到你的全局中间件中。

每当你的中间件调用 `terminate` 方法时，Laravel 会从 [服务容器](/docs/{{language}}/{{version}}/container) 中返回一个新的中间件实例。如果你想要在 `handle` 和 `terminate` 方法被调用时使用同一个中间件实例，你可以在将中间件在容器中使用 `singleton` 方法进行注册。
