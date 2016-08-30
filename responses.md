# HTTP 响应

- [构建响应](#creating-responses)
    - [附加响应头](#attaching-headers-to-responses)
    - [附加 Cookies](#attaching-cookies-to-responses)
    - [Cookies & 加密](#cookies-and-encryption)
- [重定向](#redirects)
    - [重定向至命名路由](#redirecting-named-routes)
    - [重定向至控制器动作](#redirecting-controller-actions)
    - [重定向伴随闪存数据](#redirecting-with-flashed-session-data)
- [其他响应类型](#other-response-types)
    - [视图响应](#view-responses)
    - [JSON 响应](#json-responses)
    - [文件下载](#file-downloads)
    - [文件响应](#file-responses)
- [响应宏](#response-macros)

<a name="creating-responses"></a>
## 构建响应

#### 字符串 & 数组

所有的路由和控制器都应该返回某种响应发送给用户的浏览器，Laravel 提供了多种不同的方式来返回响应。最基本的响应就是简单的在路由或控制器中返回字符串。框架会自动的将字符串进行转换并融入到完整的 HTTP 响应中：

    Route::get('/', function () {
        return 'Hello World';
    });

除了可以从路由或者控制器中返回字符串之外，你也可以返回数组。框架会自动的将数组转换为 JSON 响应：

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} 你知道你也可以在路由或控制器中直接返回 [Eloquent collections](/docs/{{language}}/{{version}}/eloquent-collections) 吗？它们会自动的被转换为 JSON。快点尝试一下!

#### 响应对象

大多数时候，路由或者控制器的动作应该返回 `Illuminate\Http\Response` 实例或者 [视图](/docs/{{language}}/{{version}}/views)。

返回一个完整的 `Response` 实例允许你自由的修正响应的 HTTP 状态和请信息。一个 `Response` 实例继承自 `Symfony\Component\HttpFoundation\Response` 类，该类提供了多种方法来生成一个 HTTP 响应：

    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="attaching-headers-to-responses"></a>
#### 附加响应头

`Response` 实例的大多数方法都是允许进行链式调用的，你可以使用链式调用来流利的构建响应。例如，你可以在响应被发送到用户之前使用 `header` 方法去附加多种响应头:

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

或者你可以使用 `withHeaders` 方法来指定一个包含了多个响应头的数组附加到响应中：

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="attaching-cookies-to-responses"></a>
#### 附加 Cookies

`Response` 实例的 `cookie` 方法可以允许你简单的追加 cookie 信息到响应中。例如，你可以使用 `cookie` 方法来生成一个 cookie 并附加到响应中：

    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);

`cookie` 方法允许你添加额外的参数来一进步定制化你的 cookie 属性，通常，这些参数的功能与原生 PHP 的 [setcookie](http://php.net/manual/en/function.setcookie.php) 方法一致：

    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

<a name="cookies-and-encryption"></a>
#### Cookies & 加密

默认的，Laravel 中所有的 cookies 都是经过签证加密的，所以客户端的用户是没有办法进行修改或解读的。如果你想要在生成某些 cookie 时，禁用默认的加密，那么你需要在 `App\Http\Middleware\EncreyptCookies` 中间件的 `$except` 属性中进行追加，它被存放在 `app/Http/Middleware` 目录中：

    /**
     * The names of the cookies that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## 重定向

重定向应是一个 `Illumiante\Http\RedirectResponse` 类的实例，它包含了所有重定向到指定 URI 所需要的响应头信息。Laravel 提供了多种方式去生成重定向实例。最简单的方式莫过于使用全局帮助方法 `redirect`：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有时候你需要将用户重定向到上一次请求的地址，那么你可以使用全局帮助方法 `back`。因为这里用到了 [session](/docs/{{language}}/{{version}}/session) 中间件，默认的 Laravel 路由都被包裹了 `web` 中间件组中，`web` 中间件组中已经包含了 session 中间件:

    Route::post('user/profile', function () {
        // Validate the request...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### 重定向至命名路由

当你调用 `redirect` 帮助方法而不传递任何参数是，它将返回一个 `Illuminate\Routing\Redirector` 的实例，这个实例允许你使用一些方法来处理一些重定向的信息。例如，为了生成一个命名路由的 `RedirectResponse` 实例，你可以使用 `route` 方法:

    return redirect()->route('login');

如果命名路由也含有其它参数，那么你可以传递第二个参数到 `route` 方法：

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 通过 Eloquent 模型填充参数

如果你需要重定向的路由是使用 `Eloquent` 模型的 `ID` 作为参数识别的路由，那么你可以直接在 `route` 方法中传递用户实例，它会自动被解析到 ID:

    // For a route with the following URI: profile/{id}

    return redirect()->route('profile', [$user]);

如果你希望定制化路由参数中的值，你可以重写 Eloquent 模型中的 `getRouteKey` 方法：

    /**
     * Get the value of the model's route key.
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### 重定向至控制器动作

你也可以生成重定向信息到 [控制器的动作](/docs/{{language}}/{{version}}/controllers) 中，你可以简单的通过 `action` 方法传递控制器名称和动作来做到这些。你应该注意到，你并不需要特别指出控制器的全部命名空间，因为 Larvel 的 `RouteServiceProvider` 已经自动的设置了默认的命名空间：

    return redirect()->action('HomeController@index');

如果你的控制器动作中也接收其它的参数，你同样可以在 `action` 方法中传递第二个参数：

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-with-flashed-session-data"></a>
### 重定向并闪存 Session

你可以通过 `RedirectResponse` 实例的链式调用来生成重定向信息的同时 [闪存数据到 session 中](/docs/{{language}}/{{version}}/session#flash-data)。这在执行动作之后存储消息状态的场景尤其有用，为了方便，你可以在创建 `RedirectResponse` 实例的同时链式的调用 `with` 方法来闪存数据到 session:

    Route::post('user/profile', function () {
        // Update the user's profile...

        return redirect('dashboard')->with('status', 'Profile updated!');
    });

当然，在用户重定向到新页面之后，你是可以访问到闪存的 [session](/docs/{{language}}/{{version}}/session) 信息的，例如，在 [Blade syntax](/docs/{{language}}/{{version}}/blade) 中你可以这么使用:

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="other-response-types"></a>
## 其他响应类型

帮助方法 `response` 也可以用来方便的生成其它类型的相应实例，如果你使用 `response` 帮助方法而不传递任何的参数，那么它会返回一个实现了 `Illuinate\Contracts\Routing\ResponseFactory` [契约](/docs/{{language}}/{{version}}/contracts) 的实例。该契约提供了多种有用的方法来生成响应。

<a name="view-responses"></a>
### 视图响应

如果你想控制响应的头信息和状态，并且你也需要返回一个 [视图](/docs/{{language}}/{{version}}/views) 作为响应的内容，那么你可以使用 `view` 方法：

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

当然，如果你并不需要定制化响应的状态或者头信息，那么你可以直接简单的使用全局帮助方法 `view`。

<a name="json-responses"></a>
### JSON 响应

`json` 方法会自动的设置响应头的 `Content-Type` 为 `application/json`，以及使用 `json_encode` PHP 函数将给定的数组转换为 JSON：

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA'
    ]);

如果你想要生成 `JSONP` 的响应，那么你可以使用 `json` 方法然后追加 `withCallback` 方法：

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### File Downloads

`download` 方法可以在返回一个强制用户浏览器进行下载指定路径文件的响应。`download` 方法也允许传递第二个参数作为浏览器下载时的文件名，你也可以传递 HTTP 响应头数组作为第三个参数：

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> {note} Symfony HttpFoundation 需要被下载的文件有一个 ASCII 文件名。

<a name="file-responses"></a>
### File 响应

`file` 方法允许你在浏览器直接显示如 image 或者 pdf 类型的文件，从而替代直接下载。该方法接收文件路径作为第一个参数，也可以接收 HTTP 响应头数组作为第二个参数:

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="response-macros"></a>
## 响应宏

如果你想为你的控制器或者路由定义某种可复用的响应，那么你可以使用 `Response` 假面的 `macro` 方法。比如，在 [service provider's](/docs/{{language}}/{{version}}/providers) 的 `boot` 方法中:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Response;

    class ResponseMacroServiceProvider extends ServiceProvider
    {
        /**
         * Register the application's response macros.
         *
         * @return void
         */
        public function boot()
        {
            Response::macro('caps', function ($value) {
                return Response::make(strtoupper($value));
            });
        }
    }

`macro` 方法接收一个别名作为第一个参数，接收一个闭包作为第二个参数。闭包会在访问 `ResponseFactory` 实现的动态属性 `macro` 别名时执行，或者通过全局帮助方法 `response` 调用别名时执行:
The `macro` function accepts a name as its first argument, and a Closure as its second. The macro's Closure will be executed when calling the macro name from a `ResponseFactory` implementation or the `response` helper:

    return response()->caps('foo');
