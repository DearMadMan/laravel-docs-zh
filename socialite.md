# Laravel Socialite

- [前言](#introduction) 
- [Open Authorization](#open-authorization)
- [安装](#installation)
- [集成微信](#wechat-extend)

<a name="installation"></a>
## 前言

Laravel Socialite provides an expressive, fluent interface to OAuth authentication with Facebook, Twitter, Google, LinkedIn, GitHub and Bitbucket. It handles almost all of the boilerplate social authentication code you are dreading writing.

[Laravel Socialite](https://github.com/laravel/socialite) 为第三方应用的 OAuth 认证提供了非常丰富友好的接口，我们使用它可以非常方便快捷的对类似微信、微博等第三方登录进行集成。

<a name="open-authorization"></a>
## Open Authorization

OAuth（开放授权）是一个开放标准，允许用户让第三方应用访问该用户在某一网站上存储的私密的资源（如照片，视频，联系人列表），而无需将用户名和密码提供给第三方应用。

OAuth 允许用户提供一个令牌，而不是用户名和密码来访问他们存放在特定服务提供者的数据。每一个令牌授权一个特定的网站（例如，视频编辑网站)在特定的时段（例如，接下来的 2 小时内）内访问特定的资源（例如仅仅是某一相册中的视频）。这样，OAuth 让用户可以授权第三方网站访问他们存储在另外服务提供者的某些特定信息，而非所有内容。

我们的应用与使用 OAuth 标准的第三方应用的交互流程一般是这样的：

- 展示跳转到第三方应用登录的链接
- 用户点击链接跳转到第三方应用登录并进行授权
- 在用户授权后第三方应用会跳转到我们所指定的应用回调资源地址并伴随用于交互 AccessToken 的 Code
- 我们的应用拿到 Code 后自主请求第三方应用并使用 Code 获取该用户的 AccessToken
- 获取 AccessToken 之后的应用即可自主的从第三方应用中获取用户的资源信息

<style>
    #socialite-uml img {
        max-width: 100%;
    }
</style>

<div id="socialite-uml" markdown="1">

### Laravel Socialite UML

![Socialite](/assets/img/socialite.jpg)

</div>

<a name= installation></a>
## 安装 Laravel Socialite

使用 Composer 进行安装：

```php
composer require laravel/socialite
```

### 配置

你需要在 `config/app.php` 的 `providers` 键中追加：

```php
'providers' => [
    // Other service providers...

    Laravel\Socialite\SocialiteServiceProvider::class,
],
```

在 `aliasses` 键中添加 `Socialite`:

```php
'Socialite' => Laravel\Socialite\Facades\Socialite::class,
```

在 `config/services.php` 配置文件中添加驱动器配置项：

```php
'github' => [
    'client_id' => 'your-github-app-id',
    'client_secret' => 'your-github-app-secret',
    'redirect' => 'http://your-callback-url',
],
```

至此整个流程安装完毕。

<a name="wechat-extend"></a>
## 集成微信登录

集成微信我们需要提供一个 `WechatServiceProvider` 和 一个 `WechatProvider`，用这两个文件来为微信登录提供驱动，Laravel Socialite 的 SocialiteManager 继承自 `Illuminate\Support\Manager` 类，而其对自定义驱动提供了友好的接口支持，所以我们可以手动的添加一个 Wechat 驱动器：

```php
<?php

namespace Crowdfunding\Providers\Socialite;

use Laravel\Socialite\Two\AbstractProvider;
use Laravel\Socialite\Contracts\Provider as ProviderInterface;
use Laravel\Socialite\Two\User;
use GuzzleHttp\ClientInterface;

class WechatProvider extends AbstractProvider implements ProviderInterface
{
    /**
    * openid for get user.
    * @var string
    */
    protected $openId;

    /**
     * set Open Id.
     *
     * @param  string  $openId
     */
    public function setOpenId($openId) {
        $this->openId = $openId;

        return $this;
    }

    /**
     * {@inheritdoc}.
     */
    protected $scopes = ['snsapi_login'];

    /**
     * {@inheritdoc}.
     */
    public function getAuthUrl($state)
    {
        return $this->buildAuthUrlFromBase('https://open.weixin.qq.com/connect/qrconnect', $state);
    }

    /**
     * {@inheritdoc}.
     */
    protected function buildAuthUrlFromBase($url, $state)
    {
        $query = http_build_query($this->getCodeFields($state), '', '&', $this->encodingType);

        return $url.'?'.$query.'#wechat_redirect';
    }

    /**
     * {@inheritdoc}.
     */
    protected function getCodeFields($state = null)
    {
        return [
            'appid'         => $this->clientId, 'redirect_uri' => $this->redirectUrl,
            'response_type' => 'code', 'scope'                 => $this->formatScopes($this->scopes, $this->scopeSeparator),
            'state'         => $state,
        ];
    }

    /**
     * {@inheritdoc}.
     */
    public function getTokenUrl()
    {
        return 'https://api.weixin.qq.com/sns/oauth2/access_token';
    }

    /**
     * {@inheritdoc}.
     */
    public function getUserByToken($token)
    {
        $response = $this->getHttpClient()->get('https://api.weixin.qq.com/sns/userinfo', [
            'query' => [
                'access_token' => $token,
                'openid'       => $this->openId,
                'lang'         => 'zh_CN',
            ],
        ]);

        return json_decode($response->getBody(), true);

    }

    /**
     * {@inheritdoc}.
     */
    public function mapUserToObject(array $user)
    {
        return (new User())->setRaw($user)->map([
          'openid' => $user['openid'], 'nickname' => $user['nickname'],
          'avatar' => $user['headimgurl'], 'name' => $user['nickname'],
          'email'  => null, 'unionid'             => $user['unionid']
        ]);
    }

    /**
    * {@inheritdoc}.
    */
    protected function getTokenFields($code)
    {
        return [
            'appid' => $this->clientId, 'secret' => $this->clientSecret,
            'code' => $code, 'grant_type' => 'authorization_code',
        ];
    }

    /**
    * {@inheritdoc}.
    */
    public function getAccessTokenResponse($code)
    {
        $postKey = (version_compare(ClientInterface::VERSION, '6') === 1) ? 'form_params' : 'body';

        $response = $this->getHttpClient()->post($this->getTokenUrl(), [
            'headers' => ['Accept' => 'application/json'],
            $postKey => $this->getTokenFields($code),
        ]);

        $responseBody = json_decode($response->getBody(), true);
        $this->setOpenId($responseBody['openid']);

        return $responseBody;
    }
}

```

编写完驱动之后我们需要注册该驱动器到 SocialiteManager 中，因此我们编写一个 WechatServiceProvider:

```php
<?php

namespace Crowdfunding\Providers\Socialite;

use Illuminate\Support\ServiceProvider;

class WechatServiceProvider extends ServiceProvider
{
    public function boot()
    {
       $this->app->make('Laravel\Socialite\Contracts\Factory')->extend('wechat', function ($app) {
            $config = $app['config']['services.wechat'];
            return new WechatProvider(
                $app['request'], $config['client_id'],
                $config['client_secret'], $config['redirect']
            );
       });
    }
    public function register()
    {

    }
}

```

接着我们就可以添加配置项及将服务提供者注册到 Laravel 中：

```php
// app.php
'providers' => [
    // Other service providers...
    Crowdfunding\Providers\Socialite\WechatServiceProvider::class,
],

// services.php
'wechat' => [
    'client_id' => 'appid',
    'client_secret' => 'appSecret',
    'redirect' => 'http://xxxxxx.proxy.qqbrowser.cc/oauth/callback/driver/wechat',
]
```

紧接着添加路由及控制器：

```php
// route.php
Route::group(['middleware' => 'web'], function () {
    Route::get('oauth/callback/driver/{driver}', 'OAuthAuthorizationController@handleProviderCallback');
    Route::get('oauth/redirect/driver/{driver}', 'OauthAuthorizationController@redirectToProvider');
});
```

控制器：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

use App\Http\Requests;
use Socialite;

class OAuthAuthorizationController extends Controller
{
    //
    public function redirectToProvider($driver) {
        return Socialite::driver($driver)->redirect();
    }

    public function handleProviderCallback($driver) {
        $user =  Socialite::driver($driver)->user();
        // dd($user)
    }

}

```

至此集成完毕。

PS: 欢迎关注简书 [Laravel](http://www.jianshu.com/collection/3dff40fa5135) 专题，也欢迎 Laravel 相关文章的投稿 :)，作者知识技能水平有限，如果你有更好的设计方案欢迎讨论 :)