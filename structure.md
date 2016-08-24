# Directory Structure

- [前言](#introduction)
- [根目录](#the-root-directory)
    - [`app` 目录](#the-root-app-directory)
    - [`bootstrap` 目录](#the-bootstrap-directory)
    - [`config` 目录](#the-config-directory)
    - [`database` 目录](#the-database-directory)
    - [`public` 目录](#the-public-directory)
    - [`resources` 目录](#-resources-directory)
    - [`routes` 目录](#the-routes-directory)
    - [`storage` 目录](#the-storage-directory)
    - [`tests` 目录](#the-tests-directory)
    - [`vendor` 目录](#the-vendor-directory)
- [App 目录](#the-app-directory)
    - [`Console` 目录](#the-console-directory)
    - [`Events` 目录](#the-events-directory)
    - [`Exceptions` 目录](#the-exceptions-directory)
    - [`Http` 目录](#the-http-directory)
    - [`Jobs` 目录](#the-jobs-directory)
    - [`Listeners` 目录](#the-listeners-directory)
    - [`Mail` 目录](#the-mail-directory)
    - [`Notifications` 目录](#the-notifications-directory)
    - [`Policies` 目录](#the-policies-directory)
    - [`Providers` 目录](#the-providers-directory)

<a name="introduction"></a>
## 前言

默认的 Laravel 应用目录结构旨在为大型和小型应用提供一个很好的起点。当然你可以自主的去管理你的应用目录。Laravel 并没有限制类应该在哪里被构建，只要它能被 Composer 自动加载就行。

#### 模型目录在哪呢？

当刚开始学习 Laravel 时，很多开发者都会有疑惑，`models` 目录在哪儿？事实上，Laravel 是有意缺失模型目录的。我们发现对于不同的人来说 "models" 是一个模棱两可的概念，它代表了太多不同的东西。有些开发引用一些框架中的 "model"，将所有的业务逻辑全部塞了进去，而有些开发者参考其它的框架中的 "models" 将所有与关联数据库交互的能力封装成模型类。

出于这个原因，我们选择将 `app` 作为 Eloquent 模型的默认目录，当然你选择将其存放在其它地方也是可以的。

<a name="the-root-directory"></a>
## 根目录

<a name="the-root-app-directory"></a>
#### App 目录

`app` 目录，就如你所预料的那样，这里包含了应用中的核心代码，我们将很快会探索这个目录的详情。不管怎么样，在你的应用中的大多数类都被存放在这个目录中。

<a name="the-bootstrap-directory"></a>
#### Bootstrap 目录

`bootstrap` 目录包含了一些文件去做整个框架的引导启动和配置的自动加载，这个目录下还包含了为了提高应用的启动性能而自动生成的缓存文件存放的目录 `cache`，路由和服务缓存都会存放在这个目录里。

<a name="the-config-directory"></a>
#### Config 目录

`config` 目录，如同名字所暗示的那样，这里包含了应用中各种服务的配置文件。你可以通过浏览这个目录里的文件来熟悉框架中的一些可用的选项，这是非常棒的策略。

<a name="the-database-directory"></a>
#### Database 目录

`database` 目录包含应用中所有的数据库迁移文件和种子文件，如果你需要，你也可以使用这个目录在存放 SQLite 数据库。

<a name="the-public-directory"></a>
#### Public 目录

`public` 目录包含了所有前端控制器和静态资源文件，比如图片，JavasScript，CSS，等等。其中 `index.php` 是应用中所有请求的入口。

<a name="the-resources-directory"></a>
#### Resources 目录

`resources` 目录包含了所有的视图，原始未被压缩编译的资源文件（LESS，SASS，CoffeeScript，或 JavaScript）。这个目录下同时也存放了所有的语言文件。

<a name="the-routes-directory"></a>
#### Routes 目录

`routes` 目录包含了所有应用中定义的路由。默认的，Laravel 框架中引用了两个路由文件: `web.php` 和 `api.php`。其中 `web.php` 文件中的路由在被 `RouteServiceProvider` 引入时分配了 `web` 中间件组，这个中间件组提供了如会话状态，CSRF 保护和 cookie 加密的服务。如果你的应用中并不提供那些无状态的 ResTful API，那么你所有的路由都应该被定义在 `web.php` 文件中。

而 `api.php` 文件中的路由在被 `RouteServiceProvider` 引入时放置了 `api` 中间件组，这个中间件组提供了一些访问速率的限制。这些路由被定义为无状态的，所以请求是需要进行 token 验证才能通过路由进入应用的，而不是通过访问 session 状态。

<a name="the-storage-directory"></a>
#### Storage 目录


`storage` 目录包含了所有编译了的 Blade 模板，session 文件，缓存文件和一些框架自动生成的其它文件。这个目录下分离出了 `app`，`framework` 和 `logs` 文件夹。`app` 目录可以用来存储任意对你应用有用的文件。`framework` 目录用来存储由框架生成的文件和缓存。最后，`logs` 目录包含了应用日志文件。

其中 `storage/app/public` 目录可以用来放置一些用户生成的文件，比如用户头像，这些应该都是应该具有可读的权限的。你应该创建一个 `public/storage` 软连接来指向到这个目录。你可以直接使用 `php artisan storage:link` 命令来这么做。

<a name="the-tests-directory"></a>
#### Tests 目录

`test` 目录包含了所有自动化测试文件。这里 提供了一个基于 [PHPUnit](https://phpunit.de/) 的测试用例。所有的测试类都应该以 `Test` 作为后缀。你可以使用 `phpunit` 或者 `php vendor/bin/phpunit` 命令来进行你的测试。

<a name="the-vendor-directory"></a>
#### Vendor 目录

`vendor` 目录包含了所有 [Composer](https://getcomposer.org) 加载的依赖。

<a name="the-app-directory"></a>
## App 目录

应用相关的多数文件都存储在 `app` 目录中。默认的，该目录使用的是全局命名空间 `App`，并且通过 Composer 执行 [PSR-4 自动加载标准](http://www.php-fig.org/psr/psr-4/) 进行自动加载。


`app` 目录下附带了多个子目录，如 `Console`，`Http`，和 `Providers` 等。`Console` 和 `Http` 目录提供了进入应用核心的 API。HTTP 协议和 CLI 是两种均可以和应用交互的机制，但是它们实际上并不包含应用逻辑。换句话说就是它们只是简单的向应用发布命令的两种途径。`Console` 目录包含了所有可用的 Artisan 命令，而 `HTTP` 目录包含了你的控制器，中间件和请求文件。

在 `app` 目录中的其它各种目录会在使用 `make` Artisan 命令生成类时自动生成。所以，默认新安装的应用中 `app/Jobs` 目录是不存在的，除非你执行 `make:job` Artisan 命令来生成 job 类。

> {tip} 很多类在 `app` 目录中都可以通过 Artisan 命令来生成。你可以在终端中使用 `php artisan list make` 查看所有可用的命令。

<a name="the-console-directory"></a>
#### Console 目录

`Console` 目录中包含了应用中所有自定义的 Artisan 命令。这些命令可以通过使用 `make:command` 命令来生成。同时这个目录页包含了 console 的内核，这个内核是自定义的 Artisan 命令需要注册的地方，同时它也是 [计划任务](docs/{{language}}/{{version}}/scheduling) 定义的地方。

<a name="the-events-directory"></a>
#### Events 目录

这个目录默认并不存在，但是当你使用 `event:generate` 或者 `event:make` Artisan 命令时，它会自动的别生成。`Events` 目录，如你所料的那样，这里存储 [事件类](/docs/{{language}}/{{version}}/events)。事件可以用来通知应用中其它部分给定的行为已经发生。它提供了灵活性和强大的解耦能力。

<a name="the-exceptions-directory"></a>
#### Exceptions 目录

`Exceptions` 目录包含了应用中的异常处理程序，同时这里也是一个处理应用所抛出的异常的好去处。如果你需要为异常配置定制化的日志，或者是渲染，那么你应该修改这个目录下的 `Handler` 类。

<a name="the-http-directory"></a>
#### Http 目录

`Http` 目录包含了你所有的控制器，中间件，和表单请求。所有处理请求的逻辑都应该被存放在这个目录中。

<a name="the-jobs-directory"></a>
#### Jobs 目录

这个目录默认是不包含的。但是会在你执行 `make:job` Artisan 命令时自动生成。`Jobs` 目录是存储应用中 [队列任务](/docs/{{language}}/{{version}}/queues) 的地方。应用中的任务可以被队列化或者也可以在当前请求周期内同步进行。在当前请求中进行同步任务有时也会被称作 "commands"，这是由于它们是 [command pattern](https://en.wikipedia.org/wiki/Command_pattern) 的一种实现。

<a name="the-listeners-directory"></a>
#### Listeners 目录

这个目录默认是不包含的。但是它会在你执行 `event:generate` 或者 `make:listener` Artisan 命令时自动生成。`Listeners` 目录包含了所有 [事件](docs/{{language}}/{{version}}/events) 的处理器类，处理器接收一个事件类并在事件触发时提供响应逻辑。例如，`UserRegistered` 事件可以被 `SendWelcomeEmail` 监听器处理。

<a name="the-mail-directory"></a>
#### Mail 目录

这个目录默认是不包含的。但是会在你执行 `make:mail` Artisan 命令时自动生成。`Mail` 目录包含了应用中所有代表邮件的类。邮件对象允许你将所有的构建邮件的逻辑封装在一个独立简单的类文件中，并且它可以通过使用 `Mail::send` 方法进行发送。

<a name="the-notifications-directory"></a>
#### Notifications 目录

这个目录默认是不包含的，但是它会在你执行 `make:notification` Artisan 命令时自动生成。`Notification` 目录包含了应用发送的所有的事务通知，比如简单的通知应用中发生了哪些事件。Laravel 的通知特性进行了高度的抽象化，它同时提供了多种通知渠道驱动，如邮件，Slack，SMS，或者存储到数据库中。

<a name="the-policies-directory"></a>
#### Policies 目录

这个目录默认是不包含的，但是它会在你执行 `make:policy` Artisan 命令时自动生成。`Policies` 目录包含了应用的授权策略类，策略类主要用来判断用户是否可以执行对资源的给定动作。你可以查看 [authorization documentation](/docs/{{language}}/{{version}}/authorization) 来了解更多。

<a name="the-providers-directory"></a>
#### Providers 目录

`Providers` 目录包含了应用中所有的 [服务提供者](docs/{{language}}/{{version}}/providers)。这些服务提供者通过绑定服务到服务容器中，注册事件，或者为即将到来的请求执行一些准备性的任务来为应用的启动进行服务。

在一个新的 Laravel 应用中，这个目录中已经包含了一些服务提供者。你可以自由的按需在这个目录中添加自己的提供者。
