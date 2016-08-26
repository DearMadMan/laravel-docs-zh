# 错误 & 日志

- [前言](#introduction)
- [配置](#configuration)
    - [错误详情](#error-detail)
    - [日志存储](#log-storage)
    - [日志分级](#log-severity-levels)
    - [自定义 Monolog 配置](#custom-monolog-configuration)
- [异常处理器](#the-exception-handler)
    - [Report 方法](#report-method)
    - [Render 方法](#render-method)
- [HTTP 异常](#http-exceptions)
    - [自定义 HTTP 错误页](#custom-http-error-pages)
- [日志记录](#logging)

<a name="introduction"></a>
## 前言

当你开始一个新的 laravel 项目时，你一定会需要用到对错误和异常的处理，而这些 laravel 都已经为你配置好了。`App\Exceptions\Handler` 类是应用中所有异常触发后被记录和渲染，然后后发送给用户的地方。我们将会在这篇文档中深入的去探索这个类。

另外，laravel 还集成了 [Monolog](https://github.com/Seldaek/monolog) 日志组件库，它提供了各种强大的日志处理器。Laravel 为你提供了几种处理器的配置，允许你选择使用单文件记录日志，还是多文件轮流记录日志，又或者是将错误日志写入系统日志中。

<a name="configuration"></a>
## 配置

<a name="error-detail"></a>
### 错误详情

你的应用中通过浏览器来展示的错误详情程度是通过你的 `config/app.php` 配置文件中的 `debug` 选项来进行配置的。默认的该配置项遵从 `.env` 文件中的 `APP_DEBUG` 环境变量。

对于本地开发环境，你应该设置 `APP_DEBUG` 环境变量为 `true`。在生成环境中，这个值应该总是为 `false`。如果这个值在生产环境中被设置为 `true`，那么它会有暴露敏感配置信息到终端用户的风险。

<a name="log-storage"></a>
### 日志存储


larvel 提供了几种开箱即用的日志模式：`single`，`daily`，`syslog` 和 `errorlog`。你应该修改 `config/app.php` 文件中的 `log` 选项来决定使用哪种存储机制。比如，如果你希望使用每日日期文件记录日志来替换默认的单文件记录方式。你可以简单的在 `config/app.php` 配置文件中该设置 `log` 选项的值：

    'log' => 'daily'

#### 日志文件最大值

当使用 `daily` 日志模式时，laravel 默认只会保留 5 天内的日志文件。如果你需要保留更多的日志文件，你需要在 `config/app.php` 文件中添加 `log_max_files` 选项：

    'log_max_files' => 30

<a name="log-severity-levels"></a>
### 日志分级

当使用 Monolog，日志消息可以依据严重程度分成不同的级别。默认的 Laravel 会存储所有级别的日志。但是，在生产环境下，你或许希望只记录某些级别的日志，你可以通过在 `app.php` 配置文件中的 `log_level` 来配置它。

当选项设置完毕，Laravel 会记录等级比这个等级更高级的日志。比如，设置 `log_level` 为 `error`，那么它会记录 `error`，`critical`，`alert` 和 `emergency` 等级的消息：

    'log_level' => env('APP_LOG_LEVEL', 'error'),

> {tip} Monolog 识别下面几种级别的日志 - 从低到高排序：`debug`，`info`，`notice`，`warning`，`error`，`critical`，`alert`，`emergency`。

<a name="custom-monolog-configuration"></a>
### 自定义 Monolog 配置

如果你想要在应用中对 Monolog 的配置拥有完全的控制，你可以使用应用的 `configureMonologUsing` 方法。你应该在 `bootstrap/app.php` 文件中进行方法的调用，并且你应该把它放置在返回 `$app` 之前：

    $app->configureMonologUsing(function($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;

<a name="the-exception-handler"></a>
## 异常处理

<a name="report-method"></a>
### Report 方法

所有的异常都会在 `App\Exceptions\Handler` 类中进行处理。该类包含了两个方法：`report` 和 `render`。我们将会对这个两个方法进行详细的剖析。`report` 方法被用来记录异常或者发送异常到其他的服务中去，比如 [BugSnag](https://bugsnag.com/) 或者 [Sentry](https://github.com/getsentry/sentry-laravel)。默认的，`report` 方法只是简单的传递异常到异常记录的基类中。事实上，你可以自由的按照自己的希望去记录异常。

比如，如果你希望用不同的方式来记录不同类型的异常，你可以使用 PHP 的 `instanceof` 方法来进行筛选操作：

    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($e instanceof CustomException) {
            //
        }

        return parent::report($exception);
    }

#### 忽略指定类型的异常


异常处理类的 `$dontReport` 属性包含了一个需要忽略的异常类数组。这个数组中类型的异常将不会被记录。默认的 404 类型的错误就不会记录在日志文件中，你可以按需在这个数组中进行追加忽略的异常类名。

    /**
     * A list of the exception types that should not be reported.
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];

<a name="render-method"></a>
### Render 方法


`render` 方法用来将异常转换为传递回浏览器的响应信息。默认的，异常会被传递到基类并返回一个响应。事实上，你可以自由的按需进行定制化响应：

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        if ($exception instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $exception);
    }

<a name="http-exceptions"></a>
## HTTP 异常

有些异常是直接从服务端来描述 HTTP 的错误代码。比如，这可能是一个“页面没有找到”的错误（404），一个“未授权”的错误（401）或者是一个“开发错误”（500）。为了在你的应用中快速的生成这种类型的响应，你可以使用 `abort` 帮助方法：

    abort(404);

`abort` 方法会立即的提出一个异常，该异常会在异常处理中被渲染。你也可以在该方法中提供一个可选的响应文本：

    abort(403, 'Unauthorized action.');

<a name="custom-http-error-pages"></a>
### 自定义 HTTP 错误页面

Laravel 使根据 HTTP 响应状态码来创建自定义的错误页面非常简单。比如，你可能希望自定义一个错误页面来提供给 404 HTTP 状态码。你可以创建一个 `resoucres/views/errors/404.blade.php` 文件。该文件会在应用抛出 404 错误时自动的提供服务。这里提供的视图名称应该是和 HTTP 状态码是相呼应的。同时 `HttpException` 实例会通过 `abort` 方法抛出，并且会被设置为 `$exception` 变量传递给视图。


<a name="logging"></a>
## 记录日志

Laravel 的日志系统是基于强大的 [Monolog](http://github.com/seldaek/monolog) 类库的。默认的，Laravel 设置了 `storage/logs` 目录来存放日志文件。你可以使用 `Log` [假面](/docs/{{language}}/{{version}}
-/facades) 来记录日志信息：

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Support\Facades\Log;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }


日志记录器根据 [RFC 5424](http://tools.ietf.org/html/rfc5424) 规范定义了 8 中日志等级：**emergency**,**alert**,**critical**,**error**,**warning**,**notice**,**info** 和 **debug**。

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

#### 上下文信息

你可以在日志方法中传递一个上下文数据的数组，这个上下文数据将会被格式化并在日志信息中显示：

    Log::info('User failed to login.', ['id' => $user->id]);

#### 访问底层的 Monolog 实例


Monolog 拥有多种额外的日志处理方法。如果你需要，你可以在 laravel 中使用下面的方式访问底层的 Monolog 实例：

    $monolog = Log::getMonolog();
