# 密码重置

- [前言](#introduction)
- [数据库考量](#resetting-database)
- [路由](#resetting-routing)
- [视图](#resetting-views)
- [重置之后](#after-resetting-passwords)
- [定制化](#password-customization)

<a name="introduction"></a>
## 前言

> {tip} **希望更快速的开始?** 你只需要在一个新的 Laravel 程序中执行 `php artisan make:auth` 命令然后在你的浏览器中访问 `http://your-app.dev/register` 或者其它你分配到应用中的 URL。这个单独的命令会自动的生成完整的认证系统脚手架，包括重置密码！

大多数的 web 应用都提供了一种用户重置其忘记密码的方式。Laravel 提供了一种便利的方式来发送密码重置邮件和提供密码重置，这样你就不需要被迫在每个应用都实现一次这种机制。

> {note} 在使用 Laravel 的密码重置特性之前，你的用户模型必须使用了 `Illuminate\Notifications\Notifiable` trait。

<a name="resetting-database"></a>
## 数据库考量

为了方便开始，请确定你的 `App\User` 模型实现了 `Illuminate\Contracts\Auth\CanResetPassword` 契约，当然，Laravel 自带的 `App\User` 模型中已经实现了这个接口，并且使用了 `Illuminate\Auth\Passwords\CanResetPassword` trait，这个性状提供了实现契约所需的方法。

#### 生成重置 Token 表迁移

接着，你必须要创建一个用来存储重置密码的 token 的表。Laravel 已经提供了这种表的迁移，并且存放在 `database/migrations` 目录下，所以，你只需要执行迁移命令：

    php artisan migrate

<a name="resetting-routing"></a>
## 路由

Laravel 自带的 `Auth\ForgotPasswordController` 和 `Auth\ResetPasswordController`  控制器包含了所有重置密码所需的逻辑。所有提供密码重置的路由都可以通过 `make:auth` Artisan 命令来生成:

    php artisan make:auth

<a name="resetting-views"></a>
## 视图

当 `make:auth` 命令被执行时，Laravel 会生成所有密码重置所需要的视图。这些视图将被存放在 `resources/views/auth/passwords` 目录中，你可以根据自己的需求去修改。

<a name="after-resetting-passwords"></a>
## 重置密码之后

一旦你生成了所有重置密码所需要的路由和视图之后，你就可以通过在浏览器访问 `/password/reset` 路由来进行密码重置。`ForgotPasswordController` 已经包含了发送重置密码的链接邮件的逻辑，而 `ResetPasswordController` 则包含了更新密码到数据库的功能。

当密码被重置之后，用户会自动登录并且被重定向到 `/home`。你可以在 `ResetPasswordController` 中定义 `redirectTo` 属性来定制化重定向地址:

    protected $redirectTo = '/dashboard';

> {note} 默认的，密码重置的 token 有效期只有一个小时，你可以在 `config/auth.php` 配置文件中修改 `expire` 选项。

<a name="password-customization"></a>
## 定制化

#### 定制化认证守卫

在你的 `auth.php` 配置文件中，你可以配置多种 "guards"，用于对各种用户表执行认证动作。你可以在 `ResetPasswordController` 重写 `guard` 方法来选择守卫，这个方法应该返回一个守卫的实例：

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 定制化密码经纪人


在你的 `auth.php` 配置文件中，你可以配置多种密码 “brokers”，用于对多种用户表进行密码重置。你可以通过在 `ForgotPasswordController` 和 `ResetPasswordController` 控制器中重写 `broker` 方法来选择指定的经纪人:

    /**
     * Get the broker to be used during password reset.
     *
     * @return PasswordBroker
     */
    protected function broker()
    {
        return Password::broker('name');
    }
