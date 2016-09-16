# 服务提供者

- [前言](#introduction)
- [编写服务提供者](#writing-service-providers)
    - [Register 方法](#the-register-method)
    - [Boot 方法](#the-boot-method)
- [注册提供者](#registering-providers)
- [惰性提供者](#deferred-providers)

<a name="introduction"></a>
## 前言

服务提供者是 Laravel 应用启动的中心。你自己的应用以及 Laravel 的核心服务都是通过服务提供者来启动的。

但是，我们所指的启动是什么意思？通常情况下，这意味着 **注册** 一些东西，包含注册服务的绑定，事件监听，中间件，和路由。服务提供者是应用配置的中心。

如果你打开 `config/app.php` 文件，你会发现 `providers` 数组。这些都是你的应用将要加载的服务提供者类。当然，很多是延迟加载的提供者，意思就是并不是所有的请求都会加载这些提供者，而是只有需要的时候才会加载。

在此篇，你将学会如果去编写自己的服务提供者并且将其注册到应用中。

<a name="writing-service-providers"></a>
## 编写服务提供者

所有的服务提供者都继承自 `Illuminate\Support\ServiceProvider` 类。大多数服务提供者都包含一个 `register` 方法和一个 `boot` 方法。在 `register` 方法中你应该**只绑定内容**到 [服务容器](/{{language}}/{{version}}/container) 。你永远不要尝试在其中注册任何的事件监听，路由或者其它功能。

Artisan CLI 可以通过 `make:provider` 命令来非常便捷的生成一个新的提供者:

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### Register 方法

就如前面所提到的，在 `register` 方法中，你应该只做一件事，那就是绑定事物到 [服务容器](/{{language}}/{{version}}/container) 中。不要做其它的事情。否则，可能你所使用的提供者提供的服务还没有被注册。

现在，让我们来看一个最基础的服务提供者。在这些服务提供者中的任意方法，你总是可以通过 `$app` 属性来访问基础的服务容器：

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection(config('riak'));
            });
        }
    }

这个服务提供者仅仅只定义了一个 `register` 方法，并且使用这个方法在服务容器中定义了一个 `Riak\Connection` 的实现。如果你并不理解服务容器是如何工作的，你可以看一下 [服务容器](/{{language}}/{{version}}/container) 的文档。

<a name="the-boot-method"></a>
### Boot 方法

那么，如果我们想在服务提供者中注册一个视图 composer 呢？那么我们应该在 `boot` 方法中做这些。`boot` **方法会在所有的服务提供者都注册完成之后才会被执行**。这意味着这时你具有访问所有服务的权限:

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->composer('view', function () {
                //
            });
        }
    }

#### Boot 方法的依赖注入

你可以在服务提供者的 `boot` 方法中使用类型提示来标识依赖。[服务容器](/{{language}}/{{version}}/container) 将会自动的将所需要的依赖注入进去：

    use Illuminate\Contracts\Routing\ResponseFactory;

    public function boot(ResponseFactory $response)
    {
        $response->macro('caps', function ($value) {
            //
        });
    }

<a name="registering-providers"></a>
## 注册提供者

所有的服务提供者都被注册在 `config/app.php` 配置文件中。这个文件包含了 `providers` 数组，这数组列出了所有服务提供者的名字。默认的，Laravel 中的核心服务都被注册在这个数组里。这些提供者启动了 Laravel 中的核心组件，如邮件，队列，缓存和其他。

你可以在该数组中进行添加注册提供者：

    'providers' => [
        // Other Service Providers

        App\Providers\ComposerServiceProvider::class,
    ],

<a name="deferred-providers"></a>
## 延迟加载提供者

如果你的提供者**只是**在 [服务容器](/{{language}}/{{version}}/container) 中注册一些绑定信息，那么你可以选择推迟注册，这样这些服务只有在真正被需要用到时才会进行注册。推迟注册能够提高你的应用的性能。因为它不会在所有请求到来时都通过文件系统来加载。

Laravel 会为哪些惰性的提供者编译并存储一张清单，只有在请求尝试解析这些服务时，Laravel 才会加载相应的提供者。

为了推迟提供者的加载，你可以在提供者类中设置 `defer` 属性为 `true` 并且定义一个 `provides` 方法。`provides` 方法应该返回提供者注册的服务容器的绑定名称:

    <?php

    namespace App\Providers;

    use Riak\Connection;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * Indicates if loading of the provider is deferred.
         *
         * @var bool
         */
        protected $defer = true;

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->app->singleton(Connection::class, function ($app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [Connection::class];
        }

    }
