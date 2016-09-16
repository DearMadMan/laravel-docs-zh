# CSRF 保护

- [前言](#csrf-introduction)
- [排除 URIs](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## 前言

对于来自 [cross-site request forgery](http://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 的攻击，Laravel 可以轻松的做出防护。跨站请求伪造是一种恶意的攻击行为，它可以绕过用户的授权去执行未授权的行为。

Laravel 会自动的为每个活跃用户生成一个 CSRF "token"，而这个 token 就是用来验证这个请求是不是真实用户自己授权的。

无论何时，当你在应用中定义 HTML 表单时，你都应该引入隐藏的 CSRF token 字段到你的表单中，这样 CSRF 保护中间件才会允许该请求通过。你可以使用 `csrf_field` 帮助函数来生成一个包含了 CSRF token 的隐藏 input 字段:

    <form method="POST" action="/profile">
        {{ csrf_field() }}
        ...
    </form>

你并不需要去手动的验证这些能够修改状态的 HTTP 请求，比如 POST，PUT，或者 DELETE 请求，已经被包含在 `web` 中间件组中的 `VerifyCsrfToken` [middleware](/{{language}}/{{version}}/middleware) 会自动的处理这些请求，它会根据请求中的 token 与会话中存储的 token 进行匹配验证。

<a name="csrf-excluding-uris"></a>
## 从 CSRF 保护中排除某些 URLs

有时候我们可能需要排除个别 URLs，让他不受 CSRF 的保护，因为可能我们需要对接一些第三方系统，这个时候系统之间的通讯是不应该使用 CSRF 机制去验证的。比如你使用了支付宝的即时付款机制，那么在用户付款的时候服务器与服务器之间会有一个回馈机制，用于支付宝通知我们的应用用户已经完成付款，这个时候我们可能需要排除 CSRF 对反馈路由的保护。

你可以在引入 `web` 中间件的路由组之外定义独立的 URLs 路由，通常来说，你不能再 `routes/web.php` 文件中定义这类路由，或者你可以在 `VerifyCsrfToken` 中的 `$except` 变量中追加 URIs 来避免 CSRF 的保护:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

    class VerifyCsrfToken extends BaseVerifier
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
        ];
    }

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Laravel 的 `VerifyCsrfToken` 中间件不仅允许通过表单中验证 `_token` 参数，它也允许通过请求头进行 CSRF 验证，它会验证请求头中的 `X-CSRF-TOKEN` 的值是否与会话中的 token 值匹配，所以你也可以这么做，存储 token 到 `meta` 标签中:

    <meta name="csrf-token" content="{{ csrf_token() }}">

当你定义了 `meta` 标签之后，你可以指导像 jQuery 类似的框架中添加 token 到所有请求的头部，这种方式可以非常简单方便的为基于 AJAX 类的应用提供 CSRF 保护：

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel 也会存储 CSRF token 到 `XSRF-TOKEN` cookie 中。你也可以在每次的请求头里引入 `X-XSRF-TOKEN` 并将其设置为 `XSRF-TOKEN` 的 cookie 值来进行 CSRF 验证。

这个 cookie 主要是为了方便一些框架而提供的，有一些 JavaScript 框架，像 Angular，会自动的为你做这些。你不太可能需要手动的使用这个值。

