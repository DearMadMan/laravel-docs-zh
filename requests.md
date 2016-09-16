# HTTP 请求

- [访问请求](#accessing-the-request)
    - [请求路径 & 方法](#request-path-and-method)
    - [PSR-7 请求](#psr7-requests)
- [检索输入](#retrieving-input)
    - [旧的输入](#old-input)
    - [Cookies](#cookies)
- [文件](#files)
    - [检索上传的文件](#retrieving-uploaded-files)
    - [存储上传的文件](#storing-uploaded-files)

<a name="accessing-the-request"></a>
## 访问请求

为了通过依赖注入能够方便的获取 HTTP 请求实例，你应该在控制器的方法中写入 `Illuminate\Http\Request` 的类型提示。当前请求的请求实例会自动的从 [服务容器](/{{language}}/{{version}}/container) 中注入：

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

#### 依赖注入 & 路由参数

如果你的控制器方法也需要接收来自路由的参数，那么你需要在进行依赖注入的参数之后添加要接收的参数。例如，你的路由是这么定义的：

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

#### 通过路由闭包访问请求实例

你可以在路由的闭包中使用 `Illuminate\Http\Request` 的类型提示。当这个路由被匹配执行时，服务容器会自动的注入当前请求的实例到路由中:

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        //
    });

<a name="request-path-and-method"></a>
### 请求路径 & 方法

`Illuminate\Http\Request` 实例为你的应用提供了多种检查 HTTP 请求的方法，它继承自 `Symfony\Component\HttpFoundation\Request` 类。我们将会在下面探讨一些重要的方法。

#### 检索请求的路径

`path` 方法可以返回请求的路径信息。所以，当请求的目标地址是 `http://domain.com/foo/bar` 时，`path` 方法将会返回 `foo/bar`:

    $uri = $request->path();

`is` 方法允许你效验所请求的路径是否匹配给定的模式。你可以使用 `*` 字符来作为通配符:

    if ($request->is('admin/*')) {
        //
    }

#### 检索请求的 URL

如果你想得到请求的完整路径，那么你可以使用 `url` 或者 `fullUrl` 方法。其中 `url` 方法会返回不带有查询字符串的 URL，而 `fullUrl` 则返回包含查询字符串的 URL:

    // Without Query String...
    $url = $request->url();

    // With Query String...
    $url = $request->fullUrl();

#### 检索请求的方式

`method` 方法可以返回 HTTP 请求的方式。你也可以使用 `isMethod` 方法来验证 HTTP 的请求是否匹配给定的字符串方式:

    $method = $request->method();

    if ($request->isMethod('post')) {
        //
    }

<a name="psr7-requests"></a>
### PSR-7 标准请求

[PSR-7](http://www.php-fig.org/psr/psr-7/) 标准为 HTTP 消息指定了一些接口，包括请求和响应。如果你想要获得 PSR-7 类型的请求实例，你需要先安装一些支持库，Laravel 使用了 **Symfony HTTP Message Bridge** 组件来转换典型的 Laravel 请求和响应为兼容 PSR-7 的实现：

    composer require symfony/psr-http-message-bridge
    composer require zendframework/zend-diactoros

当你安装完成了这些库之后，你就可以简单的在你的路由闭包中或控制器方法中使用类型提示来获取 PSR-7 请求：

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} 如果路由或控制器返回的是 PSR-7 响应的实例，那么它会自动的转换为 Laravel 响应实例并展示给框架。

<a name="retrieving-input"></a>
## 检索输入

#### 检索所有输入值

你可以使用 `all` 方法来获取所有的用户输入值，该方法返回包含所有用户输入值所组成的 `数组`：

    $input = $request->all();

#### 检索一个输入值

你可以通过 `Illuminate\Http\Request` 实例的一些方法来简便的获取用户的输入值。而且你并不需要去关心用户所使用的 HTTP 请求方式，你可以通过 `input` 方法获取到所有请求方式的值：

    $name = $request->input('name');

你也可以传递第二个参数到 `input` 方法，如果该值并不存在于请求中，将作为默认值返回：

    $name = $request->input('name', 'Sally');

当表单提交的是一些输入值是数组时，你可以使用 `.` 操作符来访问请求中的数组值：

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

#### 通过动态属性访问输入值

你可以通过 `Illuminate\Http\Request` 实例的动态属性来获取用户的输入值，如果你的应用表单中存在 `name` 字段，你可以通过下面的方式来获取该请求字段：

    $name = $request->name;

当使用动态属性时，Laravel 会首先查找请求中是否包含该值，如果不存在的话，Laravel 会检索路由中的参数是否包含了该值。

#### 检索 JSON 类型的值

当传递 JSON 类请求到你的应用时，你同样可以使用 `input` 方法来访问 JSON 数据，只要请求头的 `Content-Type` 被设置了正确的 `application/json` 值。你甚至可以通过使用 `.` 操作符深入的访问 JSON 中的数组：

    $name = $request->input('user.name');

#### 检索部分输入值

如果你只需要检索输入值数据中的一小部分，那么你可以使用 `only` 和 `except` 方法，这两个方法都可以接收一个单独的 `数组` 或者动态的参数列表作为参数：

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

#### 判断请求中是否有某值

你可以使用 `has` 方法来判断请求中是否包含了用户的某个输入值，如果该值不是空的字符串，那么 `has` 方法就会返回 `true`:

    if ($request->has('name')) {
        //
    }

<a name="old-input"></a>
### 旧的输入

Laravel 允许你在下一次请求期间保持该次请求的输入。这种特性在表单验证出错时尤其有用，它可以使你复用上一次请求进行自动的填充。如果你使用了 Laravel 的 [验证服务](/{{language}}/{{version}}/validation)，那么你不需要手动的调用它们，因为 Laravel 内置的验证机制会自动的调用它们。

#### 闪存输入到 session

`Illuminate\Http\Request` 实例的 `flash` 方法会闪存当前请求的输入到 [session](/{{language}}/{{version}}/session) 中，这样可以使应用在接受用户的下次请求时进行复用:

    $request->flash();

你也可以使用 `flasOnly` 和 `flashExcept` 方法来闪存部分请求输入到 session，这些方法常用于防止一些敏感信息如密码保存在 session 中:

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

#### 闪存输入然后跳转

一个常用的场景就是你需要连同用户的输入一起返回到上一页中，那么你可以使用 `withInput` 链式方法：

    return redirect('form')->withInput();

    return redirect('form')->withInput(
        $request->except('password')
    );

#### 检索旧的输入

你可以使用 `Request` 实例的 `old` 方法来获取上一次请求所闪存的数据。`old` 方法提供了一种便捷的方式将闪存的数据从 [session](/{{language}}/{{version}}/session) 取出及剔除:

    $username = $request->old('username');

Laravel 也提供了全局的 `old` 帮助方法。如果你需要在 [Blade 模板](/{{language}}/{{version}}/blade) 中展示旧的输入，那使用 `old` 帮助函数就方便极了。如果旧的输入中没有检索到相应的值，那么将会返回 `null`:

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### Cookies

#### 从请求中检索 Cookies

Laravel 中所有的 cookies 在被创建时都会经过一个认证码进行签证加密，这就意味着 Laravel 会验证客户端对 cookie 的修改并使之无效。你可以使用 `Illuminate\Http\Request` 实例的 `cookie` 方法来获取 `cookie` 值：

    $value = $request->cookie('name');

#### 在响应中附加一个新的 Cookie

你可以使用 `cookie` 方法来附加一个 cookie 到即将流出的 `Illuminate\Http\Response` 实例。你可以传递名称，值和 cookie 应该有效的时间到这个方法中:

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

`cookie` 方法也可以接受一些不经常使用的参数。通常这些参数具有和 PHP 原生的 [setcookie](http://php.net/manual/en/function.setcookie.php) 方法具有相同的目的和意义：

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

#### 生成 Cookie 实例

如果你需要生成一个 `Symfony\Component\HttpFoundation\Cookie` 实例，等待适当的时间传递给响应实例，那么你可以使用全局帮助方法 `cookie`。这个 cookie 除非它附加到了响应实例中，否则它不会被发送到客户端的:

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="files"></a>
### 文件

<a name="retrieving-uploaded-files"></a>
#### 获取上传的文件

你可以通过 `Illuminate\Http\Request` 实例的 `file` 方法来访问上传的文件。该方法会返回一个 `Illuminate\Http\UploadedFile` 类的实例，它继承自 PHP 的 `SplFileInfo` 类，并且提供了多种与文件交互的方法：

    $file = $request->file('photo');

    $file = $request->photo;

你可以使用 `hasFile` 方法来判断文件在请求中是否存在：

    if ($request->hasFile('photo')) {
        //
    }

#### 验证文件是否上传成功

除了检查文件是否被提供之外，你也应该使用 `isValid` 方法来检查文件是否完整的上传：

    if ($request->file('photo')->isValid()) {
        //
    }

#### 文件路径 & 扩展

`UploadedFile` 类也包含了一些方法来访问文件的完整路径和它的扩展。其中 `extension` 方法会尝试根据其内容猜测文件的扩展类型。这个扩展可能与用户从客户端传递的文件扩展不同：

    $path = $request->photo->path();

    $extension = $request->photo->extension();

#### 其它文件方法

`UploadedFile` 实例还拥有其他许多可用的方法。你可以查看 [API documentation for the class](http://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html) 来获得更多的方法信息。

<a name="storing-uploaded-files"></a>
### 存储上传的文件

为了存储上传的文件，你通常需要使用你所配置的 [文件系统](/{{language}}/{{version}}/filesystem) 之一。`UploadedFile` 类中有一个 `store` 方法会移动上传的文件到你的其中的一个磁盘，这个磁盘可以是你的本地磁盘，也可以是一些类似于 Amazon S3 的云存储。

`store` 方法接收一个路径名作为参数，这个路径表明了文件应该存储在相对于文件系统所配置的根目录的哪个路径下。并且这个路径不应该包含文件名，因为文件名会自动的根据内容的 MD5 哈希值来分配。

`store` 方法也可以接受额外的第二个参数，这个参数应该是你的磁盘名称，它会将上传的文件存储到指定的磁盘中。该方法会返回文件相对于磁盘根目录的相对路径:

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

如果你不希望文件名被自动的生成，你可以使用 `storeAs` 方法，它可以接受一个路径参数和一个文件名参数，也可以接受一个磁盘名参数：

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');
