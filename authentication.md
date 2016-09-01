# 认证

- [前言](#introduction)
    - [数据库考量](#introduction-database-considerations)
- [快速入门](#authentication-quickstart)
    - [路由](#included-routing)
    - [视图](#included-views)
    - [进行认证](#included-authenticating)
    - [检索已认证用户](#retrieving-the-authenticated-user)
    - [保护路由](#protecting-routes)
    - [登录限制](#login-throttling)
- [手动认证用户](#authenticating-users)
    - [记住用户](#remembering-users)
    - [其它认证方法](#other-authentication-methods)
- [HTTP 基础认证](#http-basic-authentication)
    - [无状态 HTTP 的认证](#stateless-http-basic-authentication)
- [添加自定义守卫](#adding-custom-guards)
- [添加自定义用户提供者](#adding-custom-user-providers)
    - [用户提供者契约](#the-user-provider-contract)
    - [可认证契约](#the-authenticatable-contract)
- [事件](#events)

<a name="introduction"></a>
## 前言

> {tip} **希望更快速的开始?** 你只需要在一个新的 Laravel 程序中执行 `php artisan make:auth` 命令然后在你的浏览器中访问 `http://your-app.dev/register` 或者其它你分配到应用中的 URL。这个单独的命令会自动的生成完整的认证系统脚手架。

Laravel 使实施认证的变得非常简单，事实上，它提供了非常全面的配置项以适应应用的业务。认证的配置文件存放在 `config/auth.php` ，这里的每个选项都提供了完善的注释文档，你可以从这里调整认证服务的行为。

在 Laravel 中，认证服务的核心是由 `guards（守卫）` 和 `providers（提供者）` 组成。守卫定义了从请求中验证用户的方式，比如说，Laravel 自带了 `session` 守卫，`session` 守卫是从所存储的会话及 cookies 中去认证请求中的用户的。

提供者则定义了从持久化存储中获取用户的方式。Laravel 自身提供了 `Eloquent` 和  `database` 查询构造器两种检索方式。当然，你也可以添加额外的提供者去做这些。

如果这听起来有些混乱，请不要担心。大多数的应用程序是根本不需要修改认证服务的默认配置信息的。

<a name="introduction-database-considerations"></a>
### 数据库考量

默认的，Laravel 在 `app` 目录下包含了 `App\User` [Eloquent model](/docs/{{language}}/{{version}}/eloquent) 模型。该模型使用了默认的 Eloquent 认证驱动。如果你的应用不是使用 Eloquent 驱动，你可以使用 `database` 认证驱动，`database` 认证驱动是基于 Laravel 的查询构造器的。

当你为 `App\User` 模型去构建数据库表结构时，你应该确认密码应该保持最少 60 个字符的长度，默认的 255 是一个比较好的选择。

另外，你需要确保 `users` 表（或等量的）中包含了一个可以为 null 的字符串列 `remember_token`，它应该具有 100 个字符的长度。这个字段是用来存储用户保持长期登录的 token 值的。你可以在迁移中使用 `$table->rememberToken()` 来快速的添加该列。

<a name="authentication-quickstart"></a>
## 快速入门

Laravel 自带了多个预构建认证相关的控制器，它们存储在 `App\Http\Controllers\Auth` 命名空间下。`RegisterController` 用来处理新用户注册， `LoginController` 处理认证相关，`ForgotPasswordController` 用来为重置密码的用户发送邮件，`ResetPasswordController` 包含了重置密码的相关逻辑。它们每个控制器都是通过引入 trait 来包含它们所需要的方法。对于大多数应用来说，你是完全没有必要去修改这些控制器的。

<a name="included-routing"></a>
### 路由

laravel 提供了一个快速的方法来生成认证的路由和视图的脚手架，你可以使用 artisan 命令:

    php artisan make:auth

在一个新的应用中，这个命令会用来进行安装注册和登录的视图，也会注入所有的认证相关的路由。`HomeController` 也会被生成。这个控制器提供了 post 登录请求的处理方法。你可以根据自己的需求删除或修改这个控制器。

<a name="included-views"></a>
### 视图

就如上面所提到的，`php artisan make:auth` 命令会创建所有认证相关的视图，并且保存在 `resources/views/auth` 目录下。

`make:auth` 命令也会创建 `resoucres/views/layouts` 目录，并在该目录下为应用创建了一个基本的布局。所有的这些视图都是使用了 Bootstrap CSS 框架，你可以根据自身的需求去定制化修改。

<a name="included-authenticating"></a>
### 进行认证

现在你已经具有了认证相关的控制器、路由和视图，你的应用已经具备了注册和认证的能力。你可以通过浏览器访问你的应用，这是因为认证控制器已经包含了所有的认证用户和存储用户到数据库的方法（通过 traits）。

#### 自定义路径

当用户通过认证后会被重定向到 `/home` URL。你可以通过在 `LoginController`，`RegisterController` 和 `ResetPasswordController` 控制器中定义 `redirectTo` 属性来修改重定向地址:

    protected $redirectTo = '/';

当用户没有认证成功，它会重定向回登录地址。

#### 自定义守卫

你也可以定制化 `guard` 用来认证和注册用户。你需要在 `LoginController`，`RegisterController` 和 `ResetPasswordController` 中定义 `guard` 方法，这个方法应该返回一个守卫的实例:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 验证 / 存储 自定义

你可能会想要在用户注册时存储一些其他必要的表单字段，进而将新增用户的信息存储到数据库中，这个时候你就需要修改 `RegisterController` 类了，这个类主管用户的创建和表单的验证。

在 `RegisterController` 类中使用 `validate` 方法来验证新用户表单与验证规则的匹配程度。你可以根据自身的需要来修改这个方法。

在 `RegisterController` 类中使用 `create` 方法用来在数据库中新增一条 `App\User` 的记录。这里使用了 [Eloquent ORM](/docs/{{language}}/{{version}}/eloquent)。你可以根据自身的需求修改这个方法。

<a name="retrieving-the-authenticated-user"></a>
### 获取已认证的用户

你可以通过 `Auth` 假面来访问已经认证的用户:

    use Illuminate\Support\Facades\Auth;

    $user = Auth::user();

另外，一旦用户经过验证，你可以通过 `Illuminate\Http\Request` 的实例来访问经过认证后的用户。你应该记得，类型提示的类会被自动的注入到你的控制器方法中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }

#### 判断当前用户是否已被认证

你可以通过 `Auth` 假面的 `check` 方法来判断当前用户是否已经被认证。如果该用户已经被认证则会返回 `true`:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} 即使可以通过使用 `check` 方法来判断用户已经通过认证，你通常也需要在用户访问某些路由或者控制器时使用中间件来过滤未被授权的用户。你可以查看 [包含路由](/docs/{{language}}/{{version}}/authentication#protecting-routes)  来了解更多。

<a name="protecting-routes"></a>
### 包含路由

[路由中间件](/docs/{{language}}/{{version}}/middleware) 可以被用来限制只有已经被认证的用户才能访问所给定的路由。Laravel 自带了 `auth` 中间件，该中间件被定义在 `Illuminate\Auth\Middleware\Authenticate`。由于这个中间件已经被注册到了你的 HTTP 核心中，所以你只需要在定义路由时附加上该中间件就可以了:

    Route::get('profile', function() {
        // Only authenticated users may enter...
    })->middleware('auth');

当然，如果你使用 [控制器](/docs/{{language}}/{{version}}/controllers) 来注册路由，你可以在控制器的构造函数中使用 `middleware` 方法来附加中间件：

    public function __construct()
    {
        $this->middleware('auth');
    }

#### 指定守卫

当你附加 `auth` 中间件到路由时，你也可以指定选择使用哪个守卫来提供对用户的认证。所指定的守卫应该与你的 `auth.php` 配置文件中的 `guards` 数组中的项相对应：

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### 登录限制

如果你使用了 Laravel 內建的 `LoginController` 类，那么它所引入的 `Illuminate\Foundation\Auth\ThrottlesLogins` trait 可以被用来限制用户尝试登陆的次数。默认的，当用户进行登陆数次失败时，其将会在一分钟内无法进行登陆。限制是根据用户的 username / e-mail 和 IP 地址来判定的。

<a name="authenticating-users"></a>
## 手动进行用户认证

当然，你没有必要一定使用 Laravel 內建的认证控制器。如果你选择删除这些认证控制器，那么你需要直接的使用 Laravel 认证类来管理用户的认证。别担心，这当然不在话下。

我们将通过 `Auth` [假面](/docs/{{language}}/{{version}}/facades) 来访问 Laravel 的认证服务，所以，我们要确保在类文件的顶部引入 `Auth` 假面。接着，让我们查看一下 `attempt` 方法：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Auth;

    class AuthController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

`attempt` 方法接收一个键值对数组来作为第一个参数。数组中的值用来在数据库中查找相匹配的用户。所以，在上面的例子中，会返回匹配到 `email` 列为 `$email` 的用户。如果用户被找到，存储在数据库中的哈希后的密码会和数组中经过哈希加密的 `password` 值进行匹配。如果两个哈希后的值匹配成功的话，就会开启一个该用户已经认证的会话。

如果认证成功，则 `attempt` 方法会返回 `true`。否则返回 `false`。

`intended` 方法用来返回给重定向器用户在登录前所想要前往的 URL 地址，该方法也接收一个参数作为所请求地址不可用时的备用地址。

### 指定额外的认证信息

如果你需要，你也可以增加一些额外条件做认证查询，比如，你需要验证被标记为 'active' 的用户：

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} 在上面的例子中 `email` 并不是必备的选项，这仅仅只是作为一个示例。你可以使用任何进行用户认证的凭证来映射数据库中的字段。

### 访问指定的守卫实例

你可以使用 `Auth` 假面的 `guard` 方法来获取你所需要的守卫实例。这使你可以在同一个应用中管理多种认证模型或用户表，并实现独立的认证。

`guard` 方法中所传递的守卫名称应该与配置文件 `auth.php` 中 guards 之一相匹配:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

### 登出

你可以使用 `Auth` 假面的 `logout` 方法来进行用户的退出，该方法将会清除用户认证的会话信息:

    Auth::logout();

<a name="remembering-users"></a>
### 记住用户

如果你想要在你的应用中提供 `记住我` 的功能，你只需要在 `attempt` 方法中传递一个布尔值作为第二个参数，这将保持用户的认证信息直到用户手动的退出登录。当然，这要求你的用户表中必须包含 `remember_token` 字段用于存储用户的 token:

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} 如果你使用 Laravel 內建的 `LoginController`，那么它所引入的 traits 已经实现了记住用户的功能了。

如果你作为被记住的用户登录的，那么你可以使用 `viaRemember` 方法来判断用户的认证是不是通过 `记住我` 的 cookie:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### 其它认证方法

#### 通过用户实例进行认证

如果你需要在应用中认证一个已经存在的用户实例，你可以使用 `login` 方法。所给定的参数必须是实现了 `Illuminate\Contracts\Auth\Authenticatable`  [contract](/docs/{{language}}/{{version}}/contracts) 的一个实例。当然，在 Laravel 中 `App\User` 模型已经实现了这个接口：

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

当然，你也可以指定所使用的守卫实例:

    Auth::guard('admin')->login($user);

#### 通过用户 ID 直接认证

你可以使用 `loginUseingId` 方法来通过 ID 认证用户的信息，该方法简单的接收一个需要进行认证的用户的主键作为参数：

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### 仅认证用户一次

`once` 方法只在当前请求中进行用户认证。不会存储会话或 cookies。这对构建无状态的 API 很有帮助。并且 `once` 方法和 `attempt` 具有相同的签证方式:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP 基础认证

[HTTP Basic Authentication](http://en.wikipedia.org/wiki/Basic_access_authentication) 提供了快速的用户认证机制而不用设立专门的登录页面。为了开始，你应该附加 `auth.basic` 中间件到你的路由。Laravel 中已经內建了 `auth.basic` [中间件](/docs/{{language}}/{{version}}/middleware)，所以你不需要去定义它:

    Route::get('profile', function() {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

一旦该中间件被附加到路由中，你每次访问该路由时都会被提示要求认证信息。默认的，`auth.basic` 中间件使用 `email` 列作为用户记录的用户名。

#### FastCGI 提示

如果你使用 PHP FastCGI，HTTP 基础认证可能无法正常工作。你可以尝试在 `.htaccess` 文件中添加如下内容：

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### 无状态的 HTTP 基本认证

你也可以在使用 HTTP 基本认证时不使用 session 保存用户的身份到 cookie。这在基于 API 的认证尤其有效。为了做到这些，你可以定义一个 [中间件](/docs/{{language}}/{{version}}/middleware)，然后调用 `onceBasic` 方法。如果 `onceBasic` 方法没有返回响应，那么请求将被进一步传递到应用：

    <?php

    namespace Illuminate\Auth\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

接着，[注册路由中间件](/docs/{{language}}/{{version}}/middleware#registering-middleware) 并且附加到路由中：

    Route::get('api/user', function() {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="adding-custom-guards"></a>
## 添加自定义的守卫

你可以在 [服务提供者](/docs/{{language}}/{{version}}/providers) 中使用 `Auth` 假面的 `extend` 方法来定义自己的认证守卫，由于 Laravel 已经自带了 `AuthServiceProvider`，所以你可以将这些代码写入到这个提供者中:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

你应该能从上面的示例中看到，你应该在 `extend` 方法中返回一个 `Illuminate\Contracts\Auth\Guard` 的实现。为了定制化守卫你需要实现这个接口所包含的方法。一旦你的守卫被定义，你就可以在 `auth.php` 配置文件的 `guards` 选项中进行配置：

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## 添加自定义的用户提供者

如果你并没有使用传统的关系数据库来存储你的用户，那么你需要提供你自己的用户提供者来扩展 Laravel。你可以通过使用 `Auth` 假面的 `provider` 方法来定义自己的用户提供者。你应该在服务提供者中调用该方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

在你使用 `provider` 方法注册完成提供者之后，你需要在 `config/auth.php` 配置文件中切换你的用户提供者为新注册的提供者。首先，定义一个 `provider` 来使用你所提供的新的驱动:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

然后，你需要在 `guards` 选项中使用该提供者：

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### 用户提供者契约

`Illuminate\Contracts\Auth\UserProvider` 的实现仅仅只是管理如何从持续存储系统中获取一个 `Illuminate\Contracts\Auth\Authenticatable` 的实现。如 MySQL，Riak，等等。这个两个接口实现了 Laravel 的持续认证机制而不需要考虑用户数据是存放在哪里了或者存放的是什么类型。

让我们来看一下 `Illuminate\Contracts\Auth\UserProvider` 契约：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById` 方法用来接收一个键来返回一个用户，如 MySQL 数据库中自增的主键 ID。被匹配的 ID 应该返回一个 `Authenticatable` 的实现。

`retrieveByToken` 方法接收用于识别用户身份的唯一标识 `$identifier` 和 `remember_token` 所存储“记住我”的 `$token`。就如上面的方法一样，该方法应该返回一个 `Authenticatable` 的实现。

`updateRememberToken` 方法使用新的 `$token` 来更新用户的 `remember_token` 字段。这个新的 token 可以是用户登录成功并且设置了 `记住我` 而分配的，也可以是用户登出之后分配的 `null`。

`retrieveByCredentials` 方法接收一个数组凭证，它会在用户尝试登录时将凭证传递给 `Auth::attemp` 方法。该方法会询问底层持续存储设备凭证的匹配情况，一般该方法会使用 `where` 条件语句来进行查询 `$credentials['username']` 类似的匹配状况。这个方法应该返回一个 `UserInterface` 的实现。**不要在这个方法里做密码验证或者认证**。

`validateCredentials` 方法应该比较所给定的用户和凭证的匹配情况。比如，这个方法或许会比较 `$user-getAuthPassword()` 和 `Hash::make` 后的 `$credentials['password']`。这个方法应该只做用户和凭证间的效验和返回比较情况的布尔值。

<a name="the-authenticatable-contract"></a>
### 可认证的（Authenticatable）契约

刚才我们探讨了 `UserProvider` 中的所有方法，现在让我们来看一看 `Authenticatable` 契约，你应该记得，提供者应该从 `retrieveById` 和 `retrieveByCredentials` 方法中返回该契约接口的实现：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

这个接口相对来说非常简单。`getAuthIdentifierName` 方法应该返回用户的主键字段的名称。`getAuthIdentifier` 方法应该返回用户的主键。在 MySQL 中，这个主键一般为自增长的主键值。`getAuthPassword` 应该返回用户的经哈希后的密码。这个接口使认证系统可以在任何用户类中运行而不用管你使用的是什么 ORM 或者其它存储抽象层。默认的，Laravel 在 `app` 目录下包含了 `User` 类，该类就实现了这个契约。所以你可以参照这个类的实现方式。

<a name="events"></a>
## 事件

Laravel 在认证进程中提供了各种各样的 [事件](/docs/{{language}}/{{version}}/events)。你可以在你的 `EventServiceProvider` 中附加监听这些事件：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],
    ];
