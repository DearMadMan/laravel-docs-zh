# 任务计划（Task Scheduling）

- [前言](#introduction)
- [定义计划](#defining-schedules)
    - [调度频率选项](#schedule-frequency-options)
    - [避免任务重叠](#preventing-task-overlaps)
- [任务输出](#task-output)
- [任务钩子](#task-hooks)

<a name="introduction"></a>
## 前言

在过去，开发者需要手动的在计划表中添加一行来录入定时执行的任务。这是非常让人头疼的，因为你不得不手动的登录远端服务器去做这些事情，它并不能在代码中有效的控制。

Laravel 的命令调度允许你在 Laravel 中流利通畅的定义你的任务计划，并且，你只需要为此在服务器中增加一个单独的定时任务条目就可以了，之后就可以在代码中进行控制任务计划的数量。 你的任务计划被定义在 `app/Console/Kernel.php` 文件的 `schedule` 方法中。在开始之前，我们先看一个简单的例子。


### 启动调度器

当使用调度器时，你只想要在你的服务器中添加下面一条 Cron 就可以了。如果你不知道如何添加 Cron 条目到你的服务器中，那么你可以考虑使用如 [Laravel Forge](https://forge.laravel.com) 的服务，它可以帮助你管理 Cron 条目：

    * * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1

该 Cron 会每分钟调用 Laravel 的命令调度来执行计划任务。Laravel 会自动的评估你的任务计划并执行到期的任务。

<a name="defining-schedules"></a>
## 定义计划

你可以在 `App\Console\Kernel` 类的 `schedule` 方法中定义所有的任务计划。在开始之前，先让我们看一个任务计划的简单例子。在这个例子中，我们会在每天的午夜执行 `Closure`。在 `Closure` 中我们会执行清除数据库表的查询语句：

    <?php

    namespace App\Console;

    use DB;
    use Illuminate\Console\Scheduling\Schedule;
    use Illuminate\Foundation\Console\Kernel as ConsoleKernel;

    class Kernel extends ConsoleKernel
    {
        /**
         * The Artisan commands provided by your application.
         *
         * @var array
         */
        protected $commands = [
            \App\Console\Commands\Inspire::class,
        ];

        /**
         * Define the application's command schedule.
         *
         * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
         * @return void
         */
        protected function schedule(Schedule $schedule)
        {
            $schedule->call(function () {
                DB::table('recent_users')->delete();
            })->daily();
        }
    }

除了可以调度 `Closure` 调用，你也可以调度 [Artisan 命令](/docs/{{version}}/artisan) 和操作系统的命令。比如，你可以使用 `command` 方法来调度一个 Artisan 命令：

    $schedule->command('emails:send --force')->daily();

你可以使用 `exec` 方法来发布一个命令到操作系统中：

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### 调度频率选项

当然，这里还有各种可以分配到任务的调度方法：

Method  | Description
------------- | -------------
`->cron('* * * * * *');`           | 执行自定义的 Cron 任务计划
`->everyMinute();`                 | 每分钟执行一次任务
`->everyFiveMinutes();`            | 每五分钟执行一次任务
`->everyTenMinutes();`             | 每十分钟执行一次任务
`->hourly();`                      | 每一小时执行一次任务
`->daily();`                       | 每天的午夜执行一次任务
`->dailyAt('13:00');`              | 每天的 13:00 执行一次任务
`->twiceDaily(1, 13);`             | 每天的 1:00 & 13:00 执行一次任务
`->weekly();`                      | 每周执行一次任务
`->montyly();`                     | 每月执行一次任务
`->monthlyOn(4, '15:00');`         | 每月的 4 号 15:00 执行一次任务
`->quarterly();`                   | 每季度执行一次任务
`->yearly();`                      | 每年执行一次任务
`->timezone('America/New_York');`  | 设置时区

这些方法可以与额外的限制相结合以创建更加细微调整的计划，比如仅在一周的某几天运行。让我们调度一个计划命令来在每周的周一执行：

    // Run once per week on Monday at 1 PM...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');

    // Run hourly from 8 AM to 5 PM on weekdays...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

下面列出了额外的计划约束条件：

Method  | Description
------------- | -------------
`->weekdays();`    | 限制在平常日内（周六、日除外）
`->sundays();`     | 限制在周日
`->mondays();`     | 限制在周一
`->tuesdays();`    | 限制在周二
`->wednesdays();`  | 限制在周三
`->thursdays();`   | 限制在周四
`->fridays();`     | 限制在周五
`->staturdays();`  | 限制在周六
`->between($start, $end);`  | 限制在 $start 和 $end 之间
`->when(Closure);` | 限制基于真值测试

#### Between 时间限制

`between` 方法可以用来限制基于一天中的某个时间段来限制任务的执行:

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

类似的，`unlessBetween` 方法可以用来排除某些时间段任务的执行：

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

#### 真值约束

`when` 方法可以基于给定的真值测试的结果来约束一个任务的执行。换种说法就是，如果给定的 `Closure` 返回 `true`，那么只要其他约束并不阻止任务的执行时，该任务就会被执行：

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

`skip` 方法刚好与 `when` 相反。如果 `skip` 方法返回 `true`，那么调度任务将不会执行：

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

当链式调用 `when` 方法时，只有在所有的 `when` 约束返回 `true` 时才会执行调度任务命令。

<a name="preventing-task-overlaps"></a>
### 避免任务重叠

默认的，如果前一个任务还在进程中，计划任务还是会再次运行的。你可以使用 `withoutOverlapping` 方法来避免这种情况的发生：

    $schedule->command('emails:send')->withoutOverlapping();

在这个例子中，`emails:send` [Artisan 命令](/docs/{{version}}/artisan) 每分钟都会被调度，但是只有在进程中没有运行该命令时才会再次执行。`withoutOverlapping` 方法对于无法确定执行时间的任务特别有效，这样就可以避免同时执行越来越多的耗时任务进而增大服务器的压力。

<a name="task-output"></a>
## 任务输出

Laravel 的任务计划提供了多种方便的方法来生成计划任务的输出。首先，你需要使用 `sendOutputTo` 方法，你可以将输出存储到文件以便之后的检查：

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

如果希望追加内容到给定的文件，你应该使用 `appendOutputTo` 方法：

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

你可以使用 `emailOutputTo` 方法来将输出发送到你选定的邮箱地址中。但是你需要注意的是，你必须先使用 `sendOutputTo` 方法将输出发送到文件中。并且，在通过邮件发送任务的输出之前，你需要先配置好 Laravel 的 [邮件服务](/docs/{{version}}/mail)：

    $schedule->command('foo')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('foo@example.com');

> {note} `emailOutputTo` 和 `sendOutputTo` 方法只能在 `command` 方法中执行，并不支持 `call` 方法的调用。

<a name="task-hooks"></a>
## 任务钩子

你可以使用 `before` 和 `after` 方法来在任务计划执行或者完成时执行特定的操作：

    $schedule->command('emails:send')
             ->daily()
             ->before(function () {
                 // Task is about to start...
             })
             ->after(function () {
                 // Task is complete...
             });

#### Pinging URLs

使用 `pingBefore` 和 `thenPing` 方法，任务调度可以自动的在任务完成之前或者之后 ping 给定的 URL。这些方法通常用来通知外部的服务。比如 [Laravel Envoyer](https://envoyer.io/)，告知其计划任务将要执行或者已经完成：

    $schedule->command('emails:send')
             ->daily()
             ->pingBefore($url)
             ->thenPing($url);

使用 `pingBefore($url)` 或者 `thenPing($url)` 方法都需要引入 Guzzle HTTP 类库。你可以通过 Composer 来进行安装：

    composer require guzzlehttp/guzzle
