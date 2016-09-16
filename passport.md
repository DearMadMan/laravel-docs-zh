# API 认证 (Passport)

- [前言](#introduction)
- [安装](#installation)
    - [前端起步](#frontend-quickstart)
- [配置](#configuration)
    - [Token 生命周期](#token-lifetimes)
    - [清理无效的 Tokens](#pruning-revoked-tokens)
- [分发访问 Tokens](#issuing-access-tokens)
    - [管理客户端](#managing-clients)
    - [请求 Tokens](#requesting-tokens)
    - [刷新 Tokens](#refreshing-tokens)
- [密码获取 Tokens](#password-grant-tokens)
    - [创建一个密码获取 Tokens 的客户端](#creating-a-password-grant-client)
    - [请求 Tokens](#requesting-password-grant-tokens)
    - [请求所有权限](#requesting-all-scopes)
- [私有访问 Tokens](#personal-access-tokens)
    - [创建一个私有访问 Tokens 客户端](#creating-a-personal-access-client)
    - [管理私有访问 Tokens](#managing-personal-access-tokens)
- [保护路由](#protecting-routes)
    - [通过中间件](#via-middleware)
    - [传递访问 Token](#passing-the-access-token)
- [Token 权限](#token-scopes)
    - [定义权限](#defining-scopes)
    - [分配权限到 Tokens](#assigning-scopes-to-tokens)
    - [检查权限](#checking-scopes)
- [与 JavaScript 协作](#consuming-your-api-with-javascript)

<a name="introduction"></a>
## 前言

Laravel 已经能够非常简单的作出传统登录表单的认证，但是关于 APIs 呢？APIs 通常使用 tokens 来进行用户的认证，并且它们并不会在请求间维护会话状态。Laravel 使用 Laravel Passport 可以轻易的做到 API 认证。Laravel Passport 可以在几分钟内为你的应用提供完整的 OAuth2 服务的支持。Passport 的构建基于 Alex Bilbie 所维护的 [League OAuth2 server](https://github.com/thephpleague/oauth2-server)。

> {note} 这份文档假设你已经熟悉了 OAuth2 协议。如果你对 OAuth2 非常的陌生，那么在继续阅读下面的内容之前，请先考虑一下熟悉 OAuth2 相关的知识。

<a name="installation"></a>
## 安装

在开始之前，你需要先通过 Composer 安装 Passport:

    composer require laravel/passport

然后，你需要在你的 `config/app.php` 配置文件中的 `providers` 数组中注册 Passport 的服务提供者：

    Laravel\Passport\PassportServiceProvider::class,

Passport 的服务提供者会注册自己的数据库迁移目录和框架，所以你应该在注册完提供者之后进行数据库的迁移。Passport 的迁移文件会为你创建应用中所需要存储客户端和访问 tokens 所需要的表：

    php artisan migrate

接下来，你应该执行 `passport:install` 命令。这个命令会为生成安全访问 tokens 时所需要的加密 keys。另外，这条命令也会创建 "私有访问" 和 "密码授权" 客户端:

    php artisan passport:install

在执行完成这条命令之后，你需要添加 `Laravel\Passport\HasApiTokens` trait 到你的 `App\User` 模型中。这个性状会为你的模型提供一些有用的帮助函数，这会允许你的模型可以检查认证用户的 token 和权限：

    <?php

    namespace App;

    use Laravel\Passport\HasApiTokens;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

然后，你应该在你的 `AuthServiceProvider` 文件中的 `boot` 方法中调用 `Passport::routes` 方法。这个方法会为分发访问 tokens 和 撤销访问 tokens，客户端，私有访问 tokens 注册所必需的路由：

    <?php

    namespace App\Providers;

    use Laravel\Passport\Passport;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            'App\Model' => 'App\Policies\ModelPolicy',
        ];

        /**
         * Register any authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Passport::routes();
        }
    }

最后，在你的 `config/auth.php` 配置文件中，你应该设置 `driver` 选项的 `api` 认证守卫值为 `passport`。这会指导你的应用使用 `TokenGuard` 来对流入的 API 请求进行认证:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="frontend-quickstart"></a>
### 前端起步

> {note} 为了使用 Passport Vue 组件，你必须使用 [Vue](https://vuejs.org) JavaScript 框架。这些组件也使用 Bootstrap CSS 框架。事实上，即使你不使用这些工具，这些组件服务对于你自己的前端集成也是非常有参考价值的。

Passport 自带了 JSON API。这可以允许你的用户创建客户端和私有访问 tokens。但是，它可能会消耗一些时间来编写前端来与这些 APIs 进行交互。所以，Passport 也包含了一个预建的 [Vue](https://vuejs.org) 组件，你可以将其作为一个集成示例作为参考，也可以将其作为你自身实现的入口点。

你需要使用 `vendor:publish` Artisan 命令来发布 Passport Vue 组件:

    php artisan vendor:publish --tag=passport-components

被发布的组件将会被存放在 `resources/assets/js/components` 目录。当组件发布完成之后，你可以在你的 `resources/assets/js/app.js` 文件中注册它们:

    Vue.component(
        'passport-clients',
        require('./components/passport/Clients.vue')
    );

    Vue.component(
        'passport-authorized-clients',
        require('./components/passport/AuthorizedClients.vue')
    );

    Vue.component(
        'passport-personal-access-tokens',
        require('./components/passport/PersonalAccessTokens.vue')
    );

当组件被注册完成之后，你可以将这些指令放入到你应用中的一个模板中来创建客户端和私有访问 tokens：

    <passport-clients></passport-clients>
    <passport-authorized-clients></passport-authorized-clients>
    <passport-personal-access-tokens></passport-personal-access-tokens>

<a name="configuration"></a>
## 配置

<a name="token-lifetimes"></a>
### Token 生命周期

默认的，Passport 分发长存的访问 tokens，它们永远不需要被刷新。如果你希望为 token 的生命周期配置的稍微短一些，那么你可以使用 `tokensExpireIn` 和 `refreshTokensExpireIn` 方法。这些方法应该在 `AuthServiceProvider` 的 `boot` 方法中被调用:

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();

        Passport::tokensExpireIn(Carbon::now()->addDays(15));

        Passport::refreshTokensExpireIn(Carbon::now()->addDays(30));
    }

<a name="pruning-revoked-tokens"></a>
### 清理失效的 Tokens

默认的，Passport 并不会从你的数据库中清除那些已经被撤销的访问 tokens，随着时间的推移，在你的数据库中就会积累大量的 tokens。如果你希望 Passport 自动的清理这些撤销的 tokens，那么你应该在 `AuthServiceProvider` 的 `boot` 方法中调用 `pruneRevokedTokens` 方法：

    use Laravel\Passport\Passport;

    Passport::pruneRevokedTokens();

这个方法并不会立即的删除所有失效的 tokens。相反的，撤销的 tokens 会在用户请求一个新的访问 token 或者刷新一个已存在的 token 时被清除。

<a name="issuing-access-tokens"></a>
## 分发访问 Tokens

大多数开发者熟悉 OAuth2 的 authorization codes（授权码） 模式，当使用授权码时，一个客户端应用将会重定向用户到你的服务端，他们会允许或拒绝请求发布访问 token 到客户端：

<a name="managing-clients"></a>
### 管理客户端

首先，开发者需要为应用的 API 构建一个交互的应用。而这个交互的应用需要在应用中被注册为客户端。通常，这个交互应用的注册需要包含应用的名称和自身的 URL，当用户同意对客户端应用的授权之后，这个 URL 将会作为重定向地址使用。

#### `passport:client` 命令

创建一个客户端的最简单的方式就是使用 `passport:client` Artisan 命令。这个命令也可以用来为测试 OAuth2 功能而创建一个专有的客户端。当你执行 `client` 命令时，Passport 将会提示你更多关于客户端的信息，并且它会为你提供一个客户端 ID 和秘钥:

    php artisan passport:client

#### JSON API

由于你的用户并不被允许使用 `client` 命令来注册客户端，所以，Passport 提供了 JSON API 来帮助你创建客户端。这帮你远离了不得不手动的编写控制器和创建，更新，删除客户端的麻烦。

但是，你需要配合 Passport 的 JSON API 使用你自己的前端框架来为你的用户管理他们的客户端而提供一个管理台。下面，我们将浏览所有管理客户端相关的 API 入口。为了方便，我们将使用 [Vue](https://vuejs.org) 来演示构建 HTTP 请求到入口：

> {tip} 如果你并不想实现完整的前端客户端管理中心，那么你可以使用 [frontend quickstart](#frontend-quickstart) 在几分钟内构建完整的功能前端。

#### `GET /oauth/clients`

这个路由会认证的用户返回所有的客户端。这主要用于列出所有的用户的客户端，以便于他们修改或者删除:

    this.$http.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

这个路由用来创建一个新的客户端，它需要两个必要的数据：客户端的 `name` 和一个 `redirect` URL。`redirect` URL 是当用户允许或拒绝请求的授权之后所重定向的地址。

当客户端创建之后，它将会被分发客户端 ID 和客户端秘钥。这些值将会在向你的应用请求获取访问 tokens 时使用。同时，客户端创建的路由将会返回一个客户端的实例：

    const data = {
        name: 'Client Name',
        redirect: 'http://example.com/callback'
    };

    this.$http.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `PUT /oauth/clients/{client-id}`

这个路由用于更新客户端。它需要两个必要的数据：客户端的 `name` 和一个 `redirect` URL。`redirect` URL 是当用户允许或拒绝请求的授权之后所重定向的地址。这个路由将会返回更新后的客户端实例：

    const data = {
        name: 'New Client Name',
        redirect: 'http://example.com/callback'
    };

    this.$http.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/clients/{client-id}`

这个路由用于删除客户端：

    this.$http.delete('/oauth/clients/' + clientId)
        .then(response => {
            //
        });

<a name="requesting-tokens"></a>
### 请求 Tokens

#### 为授权而重定向

当你的客户端创建完毕之后，开发者可以使用它们的客户端的 ID 和秘钥来从你的应用中获取授权码和访问 token。首先，开发者的应用应该创建一个重定向请求到你的应用的 `oauth/authorize` 路由，它应该像下面一样:

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => '',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

> {tip} 注意，`/oauth/authorize` 路由已经在 `Passport::routes` 方法中被定义，所以，你并不需要手动的去定义这条路由。

#### 同意授权

当接收到授权请求时，Passport 会自动的为用户同意或拒绝授权请求而展示一个模板。如果他们同意了这个请求，那他们将会被重定向到开发者应用所指定的 `redirect_uri` 地址。`redirect_uri` 必须与创建应用时所填写的 `redirect` URL 相匹配。

如果你喜欢自定义授权批准的视图，那么你可以使用 `vendor:publish` Artsian 命令来发布 Passport 的视图，被发布的视图将会被放置在 `resources/views/vendor/passport` 目录，你可以自由的修改:

    php artisan vendor:publish --tag=passport-views

#### 使用授权码换取 Token

如果用户同意了授权的请求，他们将会被重定向到开发者应用。开发者应用应该分发一个 `POST` 请求到你的应用中来请求获取一个访问 token。这个请求应该包含一个授权码，这个授权码是用户同意授权请求时所分发的。在这个例子中，我们将使用 Guzzle HTTP 类库来构建一个 `POST` 请求：

    Route::get('/callback', function (Request $request) {
        $http = new GuzzleHttp\Client;

        $response = $http->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'client_secret' => 'client-secret',
                'redirect_uri' => 'http://example.com/callback',
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

这个 `/oauth/token` 路由会返回一个 JSON 响应，并且包含 `access_token`，`refresh_token` 和 `expires_in` 属性。`expires_in` 属性包含了访问 token 过期的秒数。

> {tip} 就像 `/oauth/authorize` 路由一样，`/oauth/token` 路由也已经在 `Passport::route` 方法中进行了定义，所以你不需要手动的去注册这条路由。

<a name="refreshing-tokens"></a>
### 刷新 Tokens

如果你的应用分发的是短期的访问 tokens，那么当 token 过期时，用户将会需要使用它们的刷新 token 来换取一个新的访问 token。刷新 token 在用户获取 token 时被一同提供了。在这个示例中，我们将使用 Guzzle Http 类库来刷新 token：

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'refresh_token',
            'refresh_token' => 'the-refresh-token',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

这个 `/oauth/token` 路由将会返回一个 JSON 响应，包含 `access_token`，`refresh_token`，和 `expires_in` 属性。`expires_in` 属性包含了访问 token 过期的秒数。

<a name="password-grant-tokens"></a>
## 密码获取 Tokens

OAuth2 的 password grant 模式允许你的第三方客户端，比如手机应用，使用邮箱地址或者用户名和密码来换取 token。这个允许你直接的分发安全的访问 token 到你的第三方应用，而不需要经历完整的 OAuth2 授权码模式的重定向流程。

<a name="creating-a-password-grant-client"></a>
### 创建一个密码授权的客户端

在你的应用可以通过密码授权分发 tokens 之前，你需要创建一个密码授权的客户端。你可以使用 `passport:client` 命令和 `--password` 选择来做这个。如果你已经执行了 `passport:install` 命令，那么你不需要再执行这条命令了:

    php artisan passport:client --password

<a name="requesting-password-grant-tokens"></a>
### 请求 Tokens

当你创建完成密码授权客户端之后，你可以使用用户的邮箱地址和密码来分发一个 `POST` 请求到 `/oauth/token` 路由来获取一个访问 token。你需要知道的是，这个路由已经在 `Passport::routes` 方法中被定义了，所以你并不需要手动的去定义这条路由。如果请求成功了，那么你将会接收到一个包含了 `access_token` 和 `refresh_token` 的 JSON 响应：

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '',
        ],
    ]);

    return json_decode((string) $response->getBody(), true);

> {tip} 请牢记，访问 token 默认都是长存的，但是你可以在需要时自由的定义 [tokens 的生命周期](#configuration)。

<a name="requesting-all-scopes"></a>
### 请求所有权限

当使用密码授权时，你可能希望为 token 授予应用所有的权限。你可以在请求时请求 `*` 权限。如果你请求 `*` 权限，那么 token 实例的 `can` 方法将总是返回 `true`。这个权限只能分配到由 `password` grant 分发的 token 上:

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'password',
            'client_id' => 'client-id',
            'username' => 'taylor@laravel.com',
            'password' => 'my-password',
            'scope' => '*',
        ],
    ]);

<a name="personal-access-tokens"></a>
## 私有访问 Tokens

有时候，你的用户可能希望分配访问 tokens 到他们自己而并不经过普通的授权码重定向流程。这允许用户通过你的应用的 UI 分发 tokens 到它们自身，这通常用于用户实验你的 API 或者作为简单分发访问 tokens 的方法。

> {note} 私有访问 tokens 总是长存的。它们的生命周期并不会在使用 `tokenExpireIn` 或者 `refreshTokensExpireIn` 方法时修改。

<a name="creating-a-personal-access-client"></a>
### 创建一个私有访问客户端

在你的应用可以分发私有访问 tokens 之前，你需要创建一个私有访问客户端。你可以使用 `passport:client` 命令和 `--personal` 选项来创建它。如果你已经使用了 `passport:install` 命令，那么你并不需要执行这条命令：

    php artisan passport:client --personal

<a name="managing-personal-access-tokens"></a>
### 管理私有访问 Tokens

当你完成私有访问客户端的创建之后，你可以为通过 `User` 模型实例的 `createToken` 方法为给定的用户分发 tokens。`createToken` 方法接收 token 的名字作为第一个参数，并且它也可以接收一个可选的 [scopes](#token-scopes) 数组作为第二个参数：

    $user = App\User::find(1);

    // Creating a token without scopes...
    $token = $user->createToken('Token Name')->accessToken;

    // Creating a token with scopes...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

#### JSON API

Passport 也为管理私有访问 tokens 提供了 JSON API。你可以配合你自己的前端来为你的用户提供管理私有 tokens 的管理台。下面，我们将会浏览所有关于管理私有访问 tokens 的 API 入口。为了方便，我们将使用 [Vue](https://vuejs.org) 来演示构建 HTTP 请求到入口：

> {tip} 如果你并不想实现完整的前端客户端管理中心，那么你可以使用 [frontend quickstart](#frontend-quickstart) 在几分钟内构建完整的功能前端。

#### `GET /oauth/scopes`

这个路由将返回应用中定义的所有 [权限](#scopes)。你可以使用这条路由来列出用户可以分配到私有访问 token 中的权限：

    this.$http.get('/oauth/scopes')
        .then(response => {
            console.log(response.data);
        });

#### `GET /oauth/personal-access-tokens`

这个路由将返回认证用户创建的所有私有访问 token。这主要用来列出用户所有的 token，以便于修改和删除：

    this.$http.get('/oauth/personal-access-tokens')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/personal-access-tokens`

这个路由用来创建一个新的私有访问 tokens。它需要两个必要数据：token 的 `name` 和应该分配到 token 的 `scopes`：

    const data = {
        name: 'Token Name',
        scopes: []
    };

    this.$http.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // List errors on response...
        });

#### `DELETE /oauth/personal-access-tokens/{token-id}`

这个路由被用来删除私有访问 tokens：

    this.$http.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## 保护路由

<a name="via-middleware"></a>
### 通过中间件

Passport 包含了一个 [认证守卫](/{{language}}/{{version}}/authentication#adding-custom-guards)，它将被用来验证流入请求中的访问 tokens。当你配置 `api` 守卫使用 `passport` 驱动之后，你只需要在需要进行验证访问 token 的路由中指定 `auth:api` 中间件即可:

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### 传递访问 Token

当访问一个被 Passport 保护的路由时，与 API 交互的应用应该指定他们的访问 token 到请求中的 `Authorization` 头中，并且作为一个 `Bearer` 形式的 token。举个例子，当使用 Guzzle HTTP 类库：

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## Token 权限


<a name="defining-scopes"></a>
### 定义权限

当请求授权访问一个账户时，权限允许你的 API 客户端请求一组特定的权限。比如，如果你构建的是一个电子商务应用，并不是所有的 API 客户端都需要下订单的能力，相反的，你可以只允许客户端只能请求授权访问订单配送的状态。换句话说就是，权限允许你应用的用户限制第三方应用的执行能力。

你可以在你的 `AuthServiceProvider` 的 `boot` 方法中使用 `Passport::tokensCan` 方法来定义你的 API 的权限。`tokensCan` 方法可以接收一个关于权限名称和权限描述所组成的数组。权限的描述可以是任意你想要描述的，它会在授权同意视图中被展示:

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

<a name="assigning-scopes-to-tokens"></a>
### 分配权限到 Tokens

#### 当请求授权码时

当使用授权码来请求访问 token 时，客户端应该指定它们所期望的权限，可以作为 `scope` 查询字符串参数。`scope` 参数应该使用空格来分割多个权限：

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### 当发行私有访问 Tokens 时

如果你使用 `User` 模型的 `createToken` 方法来发行私有访问 tokens，那么你可以传递一个期望的权限数组作为第二个参数：

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### 检查权限

Passport 包含了两个中间件来验证流入的请求是否含有请求 token，所给定的 token 是否具有给定的权限。在开始之前，你需要添加下面的中间件到你的 `app/Http/Kernel.php` 文件中的 `$routeMiddleware` 属性中:

    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,

#### 检查所有权限

`scopes` 中间件被分配到路由时会验证流入的请求中的访问 token 是否具有 *全部* 所列出的权限:

    Route::get('/orders', function () {
        // Access token has both "check-status" and "place-orders" scopes...
    })->middleware('scopes:check-status,place-orders');

#### 检查任意的权限

`scope` 中间件被分配到路由时会验证流入的请求中的访问 token 是否具有 *至少一个* 所列出的权限：

    Route::get('/orders', function () {
        // Access token has either "check-status" or "place-orders" scope...
    })->middleware('scope:check-status,place-orders');

#### 在 Token 实例中检查权限

当含有访问 token 的已经认证的请求进入到应用中之后，你仍然可以在已认证的 `User` 实例中使用 `tokenCan` 方法来检查 token 是否具有给定的权限：

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });

<a name="consuming-your-api-with-javascript"></a>
## 与 JavaScript 协作

当构建一个 API 时，它可以是非常有用的，以便能够在你的 JavaScript 应用中消耗自已的 API。这种方式开发的 API 允许你自己的应用来对接 API 和你分享到世界中的 API 一致。相同的 API 可以被你的 web 应用，手机应用，第三方应用，和任意的 SDKs 所使用。

通常的，如果你需要在你的 JavaScript 应用中使用你的 API。那么你需要在每次发出请求时都手动的附加上访问 token。事实上，Passport 为你提供了一个相关的中间件。你只需要在你的 `web` 中间件组中添加 `CreateFreshApiToken` 中间件就可以了:

    'web' => [
        // Other middleware...
        \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
    ],

这个 Passport 中间件将会在即将流出的响应中附加 `laravel_token` cookie。这个 cookie 包含了一个加密的 JSON Web Token，Passport 将会使用它来进行 API 的请求认证。现在，你可以构建到 API 的请求而不需要明确的传递访问 token 了：

    this.$http.get('/user')
        .then(response => {
            console.log(response.data);
        });

当使用这种方式进行认证时，你需要在每个请求中添加 `X-CSRF-TOKEN` 请求头来指定 CSRF token。如果你使用的是框架中默认的 [Vue](https://vuejs.org) 配置，那么 Laravel 将会自动的为你发送这个请求头：

    Vue.http.interceptors.push((request, next) => {
        request.headers['X-CSRF-TOKEN'] = Laravel.csrfToken;

        next();
    });

> {note} 如果你使用其它 JavaScript 框架，那么你需要确保在每个即将离开的请求头中配置了这个请求头。
