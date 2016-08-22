# 配置

- [前言](#introduction)
- [访问配置中的值](#accessing-configuration-values)
- [环境配置](#environment-configuration)
    - [判断当前环境](#determining-the-current-environment)
- [配置缓存](#configuration-caching)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 前言

所有的 Laravel 框架的配置文件都被存贮在 `config` 目录中。所有的配置选项都被加以文档标注，所以你可以自由的浏览这些配置文件，利用这些标注来熟悉配置选项。

<a name="accessing-configuration-values"></a>
## 访问配置中的值

你可以在应用中的任何位置使用 `config` 帮助方法来非常方便的访问配置中的值。这些配置值允许通过使用 "." 语法进行访问，它包含了你需要访问的配置文件的名称和你需要访问的配置选项。你也可以设置一个默认值以备在配置选项中不存在时返回：

    $value = config('app.timezone');

你可以通过向 `config` 帮助方法中传递一个数组的方式来在运行的过程中动态的设置配置值：

    config(['app.timezone' => 'America/Chicago']);

<a name="environment-configuration"></a>
## 环境配置

让运行基于环境的应用拥有不同的配置通常来说是非常用帮助的。比如，你可能会希望生产环境和你本地开发环境使用不同的缓存驱动。而使用基于环境的配置可以非常容易实现这种需求。

为了实现这个功能，Laravel 利用了 Vance Lucas 所开发的 [DotEnv](https::/github.com/vlucas/phpdotenv) PHP 类库。在一个新鲜的 Laravel 程序中，在你应用的根目录下应该包含一个 `.env.example` 文件。如果你通过 Composer 来安装 Laravel，那么这个文件应该会自动的被重命名为 `.env` 了。如果不是的话，那么你就需要手动的来做这件事了。

当你的应用接收到一个请求时，这个文件中所列出的所有变量都会被加载到 `$_ENV` PHP 全局变量中。不过，你可以使用 `env` 帮助方法来从这些配置文件中的变量中接收值。事实上，如果你浏览了 Laravel 的一些配置文件，你会注意到一些选项已经使用了这个帮助方法:

    'debug' => env('APP_DEBUG', false),

传递到 `env` 方法中的第二个值是一个 "默认值"。这个值会在环境变量中没有找到给定键时返回。

你的 `.env` 文件不应该被提交到你的应用源码版本控制器中，这是因为每个开发或者服务在使用你的应用的同时可能需要引入不同的环境配置。

如果你是多人的团队开发，你不妨持续的引入 `.env.example` 文件在你的应用中。并且在示例配置中放置一些占位值，这样的话，团队中的其他开发者就可以非常清晰的了解哪些环境变量是的应用运行所必需的。

<a name="determining-the-current-environment"></a>
### 判断当前环境

应用当前所处的环境是通过 `.env` 文件中的 `APP_ENV` 变量来进行判断的。你可以通过 `App` [facade](/docs{{language}}/{{version}}/facades) 的 `environment` 方法来访问这个值：

    $environment = App::environment();

你也可以传递一些参数到 `environment` 方法中来检查所处的环境与给定值是否匹配。如果确实需要，你也可以传递多个值到 `environment` 方法。如果当前环境匹配到任何给定的值，那么方法会返回 `true`:

    if (App::environment('local')) {
        // The environment is local
    }

    if (App::environment('local', 'staging')) {
        // The environment is either local OR staging...
    }

应用实例也可以通过访问 `app` 帮助方法获得：

    $environment = app()->environment();

<a name="configuration-caching"></a>
## 配置缓存

你可以使用 `config:cache` Artisan 命令来将所有的配置文件缓存到一个单独的文件中，这样你的应用可以启动的更加迅速。这会合并应用中所有的配置选项到一个独立的文件中，所以这使得框架可以进行快速的加载。

你应该经常的执行 `php artisan config:cache` 命令来作为你产品发布的一个环节。这个命令不应该在本地开发中运行，这是因为在本地开发时可能需要经常改变应用的配置信息，而你一旦忘记手动的更新配置缓存，那么将会得到意料之外的结果。

<a name="maintenance-mode"></a>
## 维护模式

当你的应用处于维护模式时，一个自定义的视图将会在任何请求到达时展示。这在进行维护和更新时可以非常轻松的禁止用户访问应用。在你的应用中的中间件栈中已经包含了一个维护模式检查。如果应用处于维护模式，那么一个 `HttpException` 异常将会被抛出，并且伴随一个 503 状态码。

你可以简单的使用 Artisan 的 `down` 命令来启用维护模式：

    php artisan down

使用 `up` 命令来关闭维护模式：

    php artisan up

#### 维护模式响应模板

默认的维护模式响应模板存放在 `resource/views/errors/503.blade.php`。你可以自由的按需修改这个视图文件。

#### 维护模式 & 队列任务

在维护模式启用期间，[queued jobs](/docs/{{language}}/{{version}}/queues) 将暂停处理。这些任务会在应用退出维护模式时继续进行处理。

#### 维护模式的可选方案

由于维护模式需要你的应用关闭那么一段时间，你可以考虑一下像 [Envoyer](https://envoyer.io) 这种不需要关闭应用的持续集成服务。
