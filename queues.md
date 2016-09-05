# 队列

- [前言](#introduction)
    - [连接 Vs. 队列](#connections-vs-queues)
    - [驱动需求](#driver-prerequisites)
- [创建任务](#creating-jobs)
    - [生成任务类](#generating-job-classes)
    - [类结构](#class-structure)
- [分派任务](#dispatching-jobs)
    - [延迟分派](#delayed-dispatching)
    - [自定义队列 & 连接](#customizing-the-queue-and-connection)
    - [错误处理](#error-handling)
- [允许队列工人](#running-the-queue-worker)
    - [队列优先级](#queue-priorities)
    - [队列工人 & 部署](#queue-workers-and-deployment)
    - [任务过期 & 超时](#job-expirations-and-timeouts)
- [Supervisor 配置](#supervisor-configuration)
- [处理失败的任务](#dealing-with-failed-jobs)
    - [清理失败的任务](#cleaning-up-after-failed-jobs)
    - [失败的任务事件](#failed-job-events)
    - [重试失败的任务](#retrying-failed-jobs)
- [任务事件](#job-events)

<a name="introduction"></a>
## 前言

Laravel 的队列服务对各种不同的后台队列服务提供了统一的 API。如 Beanstalk，Amazon SQS，Redis，甚至是关联数据库。队列允许你延迟执行消耗时间的任务，比如发送一封邮件。这样可以有效的降低请求响应的时间。

队列的配置文件被存储在 `config/queue.php` 中。在这个文件中你会发现框架所支持的队列驱动的配置连接示例。这些驱动包括：数据库，[Beanstalkd](http://kr.github.com/beanstalkd)，[Amazon SQS](http://aws.amazon.com/sqs)，[Redis](http://redis.io/)，和一个同步（本地使用）的驱动。同时，Laravel 也提供了一个 `null` 队列驱动，用于表明完全的弃用队列任务。

<a name="connections-vs-queues"></a>
### 连接 Vs. 队列

在开始探索 Laravel 队列之前，明白 "connections" 和 "queues" 间的区别是非常重要的。在你的 `config/queue.php` 配置文件中有一个 `connections` 配置选项。这个选项用来指出连接到后端的服务，比如 Amazon SQS，Beanstalk，或者 Redis。事实上，任意给定的队列连接都可以拥有多个 "队列"，它们可以被认为是包含不同队列任务的堆栈。

你需要注意在 `queue` 配置文件中的每个连接配置示例中都包含了 `queue` 属性。这个属性指明了当一个任务被分派到当前连接时所使用的默认队列。换句话说就是，如果你在分派任务时并没有明确的定义这个任务应该分派到哪个队列，那么这个任务将会被分派到当前连接配置中的 `queue` 属性所定义的队列中：

    // This job is sent to the default queue...
    dispatch(new Job);

    // This job is sent to the "emails" queue...
    dispatch((new Job)->onQueue('emails'));

对于一些应用来说，它可能会更喜欢将任务分配到一个队列中，可能永远不会使用多个队列。事实上，将一个任务推送到多个队列对于需要对应用中的任务进行分级或者分段处理时非常有用，而 Laravel 的 队列工人允许你为队列指定优先级。比如，如果你希望将任务推送到 `high` 队列，你可以执行工人并给它们更高的执行优先级：

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### 驱动需求

#### 数据库

如果使用 `database` 队列驱动，你需要添加一个数据表来处理队列任务。你可以使用 `queue:table` Artisan 命令来生成一个迁移表。一旦该迁移表生成完成，你就可以使用 `migrate` 命令将其迁移到数据库中：

    php artisan queue:table

    php artisan migrate

#### 其它驱动需求

下面列出了其它队列驱动及其相应的依赖：

<div class="content-list" markdown="1">
- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~3.0`
- Redis: `predis/predis ~1.0`
</div>

<a name="creating-jobs"></a>
## 创造任务

<a name="generating-job-classes"></a>
### 生成任务类

默认的，所有的可队列执行的任务都被存储在 `app/Jobs` 目录下，如果 `app/Jobs` 目录并不存在，它将会在你执行 `make:job` Artisan 命令时被创建。你可以使用 Artisan CLI 来生辰一个新的队列任务：

    php artisan make:job SendReminderEmail

该命令会在 `app/Jobs` 目录下生成一个新的类。该类会实现 `Illuminate\Contracts\Queue\ShouldQueue` 接口，该接口表明 Laravel 应该将该任务添加到后台的任务队列中，而不是同步执行。

<a name="class-structure"></a>
### 类结构

任务类是十分简单的，通常它只包含一个 `handle` 方法来在队列任务被执行时被调用。为了快速的开始，让我们先看一个简单的任务类示例。在这个示例中，我们假装我们在管理一个播客发布服务，并且我们需要在它们被发布前先处理所上传的播客文件：

    <?php

    namespace App\Jobs;

    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }
    }

在这个例子中，你需要注意的是，我们可以直接的在队列任务的构造函数中传送一个 [Eloquent model](/docs/{{version}}/eloquent)。因为我们引入了 `SerializesModels` trait，所以当队列任务执行时，Eloquent 模型会被优雅的序列化和反序列化。如果队列任务在构造器中接收了 Eloquent 模型，那么队列任务只会序列化模型的 ID。而在任务需要进行处理时，队列系统会从数据库中自动的根据 ID 检索出模型实例。这在应用中完全是透明的，这样就可以避免了序列化完整的模型可能在队列中出现的问题。

`handle` 方法会在队列任务执行时进行调用。你需要知道的是，我们可以在任务的 `handle` 方法中使用类型提示来进行依赖的注入。Laravel 的 [服务容器](/docs/{{version}}/container) 会自动的将这些依赖注入进去。

<a name="dispatching-jobs"></a>
## 分派任务

但你编写完成任务类之后，你可以使用 `dispatch` 帮助方法来分发任务。你只需要在 `dispatch` 帮助方法中传递任务类实例就可以了：

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            dispatch(new ProcessPodcast($podcast));
        }
    }

> {tip} `dispatch` 帮助方法提供了方便简洁的全局可访问的方法，而它同时也是及其容易被测试的。你可以浏览 [testing documentation](/docs/{{version}}/testing) 来了解更多。

<a name="delayed-dispatching"></a>
### 延迟的任务

如果你想要延迟执行队列任务，那么你可以在你的任务实例中调用 `delay` 方法。`delay` 方法来自于 `Illuminate\Bus\Queueable` trait，它会在自动生成的任务类中被自动的引入。举个示例，让我们来指定一个任务，让其在分发之后 10 分钟才被执行:

    <?php

    namespace App\Http\Controllers;

    use Carbon\Carbon;
    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            $job = (new ProcessPodcast($pocast))
                        ->delay(Carbon::now()->addMinutes(10));

            dispatch($job);
        }
    }

> {note} Amazon SQS 队列服务有最大延迟执行限制，其最多可延迟 15 分钟。

<a name="customizing-the-queue-and-connection"></a>
### 自定义队列 & 连接

#### 分配到指定队列

你可以对你的队列任务进行分类，来将任务推送到不同的队列中，你甚至是可以分配各种队列工作的优先权。这并不是推送任务到队列配置文件中所定义的不同的队列连接上，而是仅在单个连接中指定的队列进行的操作。你可以使用任务实例的 `onQueue` 方法来指定队列:

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            $job = (new ProcessPodcast($podcast))->onQueue('processing');

            dispatch($job);
        }
    }

#### 分发到特定的连接

如果你使用多个队列连接，那么你可能需要指定任务需要被分配到哪个连接的队列中。为了指定所使用的连接，你可以使用任务实例的 `onConnection` 方法：

    <?php

    namespace App\Http\Controllers;

    use App\Jobs\ProcessPodcast;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PodcastController extends Controller
    {
        /**
         * Store a new podcast.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Create podcast...

            $job = (new ProcessPodcast($podcast))->onConnection('sqs');

            dispatch($job);
        }
    }

当然，你可以链式的调用 `onConnection` 和 `onQueue` 方法来为一个任务指定它所使用的连接和队列：

    $job = (new ProcessPodcast($podcast))
                    ->onConnection('sqs')
                    ->onQueue('processing');

<a name="error-handling"></a>
### 错误处理

如果任务进行的过程中有异常被抛出。它会自动的将任务释放，同时追加任务到队列的尾端以使任务可以进行再次尝试。该任务会被持续释放尝试，除非尝试的次数超出了所设置的最大次数。你可以在在使用 `queue:work` Artisan 任务命令时添加 `--tries` 选项来设置最大可尝试次数。我们将在后续篇幅中详细的介绍 [队列工人](#running-the-queue-worker) 。

<a name="running-the-queue-worker"></a>
## 执行队列工人

Laravel 引入了队列工人，它会在任务被推送到队列中时处理一个新的任务。你可以使用 `queue:work` Artisan 命令来运行这个工人。你需要注意的是，当 `queue:work` 命令开启之后，它持续的运行，除非你手动的停止或者关闭你的终端：

    php artisan queue:work

> {tip} 为了维持 `queue:work` 进程在后台能够持续的正常运行，你应该使用一些进行监控工具，比如 [Supervisor](#supervisor-configuration) 来保证队列工人没有停止运行。

请牢记，队列工人是一个长存的进程，并且它会在内存中存储引导应用的状态。由于这个原因，它们在启动之后，并不会对之后更改的代码做出同样的改变。所以，在你部署进程期间，请确保 [重启你的队列工人](#queue-workers-and-deployment)。

#### 指定连接 & 队列

你也可以为工人指定其所执行的队列连接。而传递到 `work` 命令的连接名称应该与 `config/queue.php` 配置文件中的一个连接配置名相同:

    php artisan queue:work redis

你甚至可以为队列工人进一步指定给定连接所使用的队列。比如，如果你所有的邮件都是在 `redis` 队列连接的 `email` 队列中进行处理，你可以使用下面的命令来发布指令启动一个工人执行特定的队列:

    php artisan queue:work redis --queue=emails

<a name="queue-priorities"></a>
### 队列优先级

有时候，你可能希望对所处理的队列进行处理顺序的排序。比如，在你的 `config/queue.php` 中你可以为你的 `redis` 连接的默认 `queue` 设置为 `low`。相应的你可能会像下面一样想要将一个任务推送到 `high` 优先级的队列:

    dispatch((new Job)->onQueue('high'));

为了使所有的 `high` 队列任务可以优先处理完成之后再继续处理 `low` 队列中的任务，你可以在执行 `work` 命令时传递一个以 `,` 分割的队列列表:

    php artisan queue:work --queue=high,low

<a name="queue-workers-and-deployment"></a>
### 队列工人 & 部署

因为队列工人的守护进程是一个常驻进程。它不会在你的代码改变时进行重启。所以，你应该使用部署脚本来在代码变更时重新部署使用守护进程的队列工人，你可以通过发布 `queue:restart` 命令来优雅的重启所有的工人:

    php artisan queue:restart

该命令会优雅的指导队列完成当前的任务后死亡，所以不会存在任务的遗漏。你应该注意的是，当你执行 `queue:restart` 命令时，队列工作就会死亡，所以你应该使用一种进程管理器，比如 [Supervisor](#supervisor-configuration)，你可以配置它来自动的重启队列工作。

<a name="job-expirations-and-timeouts"></a>
### 任务到期 & 超时

#### 任务到期

在你的 `config/queue.php` 配置文件中，每个队列连接都定义了 `retry_after` 选项。这个选项
用于指定当一个任务处理超过多少秒而未完成时将其释放到队列。比如，如果 `retry_after` 的值被设置为 `90`，那么当一个任务执行了 90 秒而没有被删除的话，它将会被重新放回队列中。通常的，你应该设置 `retry_after` 的值为任务处理完成所需的最大值。

> {note} 唯一的一个不包含 `retry_after` 选项的是 Amazon SQS 连接，当使用 SQS 时，你必须从 Amazon web 控制台中配置重试的阈值。

#### 工人超时

`queue:work` Artisan command 公开了 `--timeout` 选项。`--timeout` 选项用于指定当子队列工人执行超过多少时间时，Laravel 的队列主进程应该杀死这个子进程。有时候，子队列进行可能会因为各种原因而被冻结，比如一个外部的 HTTP 调用却没有任何的响应。`--timeout` 选项会根据指定的限制时间来清除冻结的进程:

    php artisan queue:work --timeout=60

`retry_after` 配置选项和 `--timeout` CLI 选项是不同的，但是他们的协作可以确保任务不会丢失并且只成功执行一次。

> {note} `--timeout` 值应该总是比你的 `retry_after` 值短几秒钟。这样就能确保所给定的任务在被重试之前被清除掉。如果你的 `--timeout` 选项比你的 `retry_after` 配置值大，那么你的任务可能会被执行两次。

<a name="supervisor-configuration"></a>
## Supervisor 配置

#### 安装 Supervisor

Supervisor 是 Linux 操作系统的一个进程监控器，并且它可以自动的在 `queue:work` 进程失败时进行重启。你可以使用下面的命令在 Ubuntu 中安装 Supervisor:

    sudo apt-get install supervisor

> {tip} 如果配置 Supervisor 对你来说很困难，那么你可以考虑使用 [Laravel Forge](https://forge.laravel.com)，它会自动的安装并为你的 Laravel 项目配置 Supervisor。

#### 配置 Supervisor

Supervisor 配置文件通常都存储在 `/etc/supervisor/conf.d` 目录中。在这个目录中，你可以创建任意数量的配置文件来指导 supervisor 来管理监控进程。比如，让我们创建一个 `laravel-worker.conf` 文件来开始和监控 `queue:work` 进程：

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log

上面的例子中，`numprocs` 指令会指导 Supervisor 运行 8 个 `queue:work` 进程并且对其进行监控，它会自动的在其失败时进行重启。当然，你可以修改命令的 `queue:work sqs` 部分来使用你所期望的队列驱动。

#### 开启 Supervisor

一旦配置文件创建完成，你可以使用下面的命令来更新 Supervisor 的配置文件和启动进程：

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

关于更多的 Supervisor 的配置信息，请参考 [Supervisor documentation](http://supervisord.org/index.html)。

<a name="dealing-with-failed-jobs"></a>
## 处理失败的任务

由于很多事情并不能如计划中的那样进行，有时候队列任务的执行可能会失败，不要担心，它发生对我们来说是最好的！Laravel 包含了一种便捷的方式来指定任务应该重复尝试的次数。如果任务被重复执行到指定的次数，它就会被记录到 `failed_jobs` 表中。你可以使用 `queue:failed-table` 命令来生成 `failed_jobs` 表的迁移：

    php artisan queue:failed-table

    php artisan migrate

当你运行 [队列工人](#running-the-queue-worker)，你应该使用 `--tries` 选项来指定任务的最大尝试次数，如果你并没有为 `--tries` 选项指定一个值，那么任务的重试次数将不受限制：

    php artisan queue:work redis --tries=3

<a name="cleaning-up-after-failed-jobs"></a>
### 在任务失败之后清除

你可以在任务类中定义一个 `failed` 方法，这允许你在任务出现失败时来执行一些指定的清除动作。这是一个还原任务执行动作和向用户发送警告通知的好地方。当任务失败时，`Exception` 会被传递到 `failed` 方法中:

    <?php

    namespace App\Jobs;

    use Exception;
    use App\Podcast;
    use App\AudioProcessor;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class ProcessPodcast implements ShouldQueue
    {
        use InteractsWithQueue, Queueable, SerializesModels;

        protected $podcast;

        /**
         * Create a new job instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

        /**
         * Execute the job.
         *
         * @param  AudioProcessor  $processor
         * @return void
         */
        public function handle(AudioProcessor $processor)
        {
            // Process uploaded podcast...
        }

        /**
         * The job failed to process.
         *
         * @param  Exception  $exception
         * @return void
         */
        public function failed(Exception $e)
        {
            // Send user notification of failure, etc...
        }
    }

<a name="failed-job-events"></a>
### 失败的任务事件

如果你希望为任务的失败注册一个事件，那么你可以使用 `Queue::failing` 方法。这个事件是一个为你的团队发送通知的好机会，你可以使用邮件或者 [HipChat](https://www.hipchat.com)。比如，我们可以在 `AppServiceProvider` 中来为这事件附加回调函数：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Queue\Events\JobFailed;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::failing(function (JobFailed $event) {
                // $event->connectionName
                // $event->job
                // $event->exception
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="retrying-failed-jobs"></a>
### 重试失败的任务

你可以使用 `queue:failed` Artisan 命令来显示数据库 `failed_jobs` 表中所有失败的任务：

    php artisan queue:failed

`queue:failed` 命令会列出任务的 ID，连接，队列，和失败时间。任务 ID 可以用来进行失败任务的尝试。比如，你可以尝试重新执行 ID 为 `5` 的任务：

    php artisan queue:retry 5

你可以使用 `queue:retry all` 命令来重启所有的任务：

    php artisan queue:retry all

如果你希望删除失败的任务，你可以使用 `queue:forget` 命令：

    php artisan queue:forget 5

你可以使用 `queue:flush` 命令来清除所有失败的任务：

    php artisan queue:flush

<a name="job-events"></a>
## 任务事件

你可以使用 `Queue::before` 和 `Queue::after` 方法来注册一个回调在队列任务开始之前或者队列执行成功之后调用。回调为添加额外的日志，持续执行子任务或者增加统计信息等提供了非常好的机会。通常，你应该在 [服务提供者中](/docs/{{version}}/providers) 调用 `Queue` [假面](/docs/{{version}}/facades)。我们可以在 Laravel 的 `AppServiceProvider` 中定义一个任务成功执行后的事件监听，并附加一个回调到事件中：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
