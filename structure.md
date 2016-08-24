# 应用目录结构

- [前言](#introduction)
- [根目录](#the-root-directory)
- [app 目录](#the-app-directory)

<a name="introduction"></a>
## 前言

默认的 Laravel 应用目录结构旨在为大型和小型应用提供一个很好的起点。当然你可以自主的去管理你的应用目录。Laravel 并没有限制类应该在哪里被构建，只要它能被 Composer 自动加载就行。

<a name="the-root-directory"></a>
## 根目录

新安装的 Laravel 应用根目录中包含了各种子目录：

`app` 目录，就如你所预料的那样，这里包含了应用中的核心代码，我们将很快会探索这个目录。

`bootstrap` 目录包含了一些文件去做整个框架的引导启动和配置的自动加载，这个目录下还包含了为了提高应用的启动性能而自动生成的缓存文件存放的目录 `cache`。

`config` 目录，如同名字所暗示的那样，这里包含了应用中各种服务的配置文件。

`database` 目录包含应用中所有的数据库迁移文件和种子文件，如果你需要，你也可以使用这个目录在存放 SQLite 数据库。

`public` 目录包含了所有前端控制器和静态资源文件，比如图片，JavasScript，CSS，等等。

`resources` 目录包含了所有的视图，原始资源文件（LESS，SASS，CoffeeScript），和本土化文件。

`storage` 目录博涵了所有编译了的 Blade 模板，session 文件，缓存文件和一些框架自动生成的其它文件。这个目录下分离出了 `app`，`framework` 和 `logs` 文件夹。`app` 目录可以用来存储任意对你应用有用的文件。`framework` 目录用来存储由框架生成的文件和缓存。最后，`logs` 目录包含了应用日志文件。

`test` 目录包含了所有自动化测试文件。这里 提供了一个基于 [PHPUnit](https://phpunit.de/) 的测试用例。

`vendor` 目录包含了所有 [Composer](https://getcomposer.org) 加载的依赖。

<a name="the-app-directory"></a>
## App 目录

应用的正餐都被存放在 `app` 目录中。默认的，该目录使用的是全局命名空间 `App`，并且通过 Composer 执行 [PSR-4 自动加载标准](http://www.php-fig.org/psr/psr-4/) 进行自动加载。

`app` 目录下附带了多个子目录，如 `Console`，`Http`，和 `Providers` 等。`Console` 和 `Http` 目录提供了进入应用核心的 API。HTTP 协议和 CLI 是两种均可以和应用交互的机制，但是它们实际上并不包含应用逻辑。换句话说就是它们只是简单的向应用发布命令的两种途径。`Console` 目录包含了所有可用的 Artisan 命令，而 `HTTP` 目录包含了你的控制器，中间件和请求文件。

`Events` 目录，若果你所料的那样，这里存储 [事件类](/docs/{{language}}/{{version}}/events)。事件可以用来通知应用中其它部分给定的行为已经发生。它提供了灵活性和强大的解耦能力。

`Exceptions` 目录包含了应用中的异常处理程序，这里是一个处理应用所抛出的异常的好去处。

`Jobs` 目录，当然，这里是存储 [队列任务](/docs/{{language}}/{{version}}/queues) 的地方。应用中的任务可以被队列化或者也可以在当前请求周期内同步进行。

`Listeners` 目录包含了所有事件的处理器类，处理器接收一个事件类并在事件触发时提供响应逻辑。例如，`UserRegistered` 事件可以被 `SendWelcomeEmail` 监听器处理。

`Policies` 目录包含了应用的授权侧罗类，策略类主要用来判断用户是否可以执行对资源的给定动作。你可以查看 [authorization documentation](/docs/{{language}}/{{version}}/authorization) 来了解更多。

> **注意** 很多类在 `app` 目录中都可以通过 Artisan 命令来生成。你可以在终端中使用 `php artisan list make` 查看所有可用的命令。
