# 控制台命令

- [前言](#introduction)
- [编写命令](#writing-commands)
    - [生成命令](#generating-commands)
    - [命令结构](#command-structure)
    - [闭包命令](#closure-commands)
- [定义期望的输入](#defining-input-expectations)
    - [参数](#arguments)
    - [选项](#options)
    - [数组输入](#input-arrays)
    - [输入描述](#input-descriptions)
- [命令 I/O](#command-io)
    - [检索输入](#retrieving-input)
    - [输入提示](#prompting-for-input)
    - [编写输出](#writing-output)
- [注册命令](#registering-commands)
- [通过代码执行命令](#programatically-executing-commands)
    - [从其它命令中执行命令](#calling-commands-from-other-commands)

<a name="introduction"></a>
## 前言

Artisan 是 Laravel 自带的命令行工具的通讯接口。它为应用的开发提供了多种有用的协作命令。你可以使用 `list` 命令来查看所有可用的 Artisan 命令：

    php artisan list

所有的命令都提供了帮助文档，你可以在相应的命令前使用 `help` 来查看相关命令的选项和参数：

    php artisan help migrate

<a name="writing-commands"></a>
## 编写命令

除了 Artisan 自带的命令之外，Laravel 也允许你自行定义自己的命令工具，你可以将自定义的命令工具存放到 `app/Console/Commands` 目录下，你当然也可以存放在其他任何想要存放的目录，只要你所存放的位置能基于 `composer.json` 的设置进行自动加载就行。

<a name="generating-commands"></a>
### 生成命令

你可以使用 `make:command` Artisan 命令，来进行新命令工具的生成。这个命令会在 `app/Console/Commands` 目录下生成一个新的命令类。如果你的应用中不存在这个目录，请不要担心，它会在你首次执行 `make:command` Artisan 命令时自动的生成。被生成的命令类中会包含一些默认的属性设置和方法：

    php artisan make:command SendEmails

<a name="command-structure"></a>
### 命令结构

在生成命令类之后，你需要在其中填充 `signature` 和 `description` 属性。这些内容会在你使用 `list` 命令时在屏幕上显示。当你的命令被执行时，会触发 `handle` 方法，你可以在这个方法里编写相应的命令逻辑。

> {tip} 为了更好的代码重用性，保持你的命令工具的轻量和服从应用服务来完成它们的任务是一种很好的实践。在下面的示例中，你需要注意我们注入一个服务类来完成繁重的发送邮件的任务。

让我们来看一个示例命令。你应该知道我们可以在命令类的构造函数中进行依赖的注入。Laravel [服务容器](/{{language}}/{{version}}/container) 会自动的注入所有在构造函数中使用类型提示的依赖:

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### 闭包命令

基于闭包的命令也可以用于取代使用类的方式来定义命令。使用同样方式的还有路由的闭包，也可以用于取代控制器。你可以直接认为命令闭包是命令类的一种简写方式。在 `app/Console/Kernel.php` 文件中的 `commands` 方法中，Laravel 加载了 `routes/console.php` 文件：

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

尽管这个文件并没有定义 HTTP 路由，但是它在你的应用中定义了基于控制台的切入点（路由）。在这个文件中，你可以使用 `Artisan::command` 方法来定义所有基于闭包的路由。`command` 方法接收两个参数：[命令签名](#defining-input-expectations) 和一个用于接收命令参数和选项的闭包：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

这个闭包会被绑定到底层的命令类实例中，所以你具有访问底层命令类帮助方法的所有权限。

#### 类型提示的依赖

除了可以接收命令的参数和选项，命令闭包也可以对其它的依赖进行类型提示，这样你就可以从 [服务容器](/{{language}}/{{version}}/container) 中解析它们：

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### 闭包命令的描述

当定义一个基于闭包的命令时，你可以使用 `describe` 方法来添加描述到命令中。这个描述会在你执行 `php artisan list` 或者 `php artisan help` 命令时展示：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## 定义输入期望值

当我们编写命令行工具时，通常都是通过用户输入的参数或者选项来收集输入。Laravel 使这一切变的非常简单。你可以使用 `signature` 属性在你的命令中定义你所期望得到的输入。`signature` 属性允许你使用一行富有表现力的类路由的语法来定义命令行中的名称，参数和选项。

<a name="arguments"></a>
### 参数

所有用户所提供的参数和选项都被包裹在大括号内。在下面的示例中，命令行工具定义了一个 **必须** 的参数：`user`：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

你也可以使参数可选，或者为一个可选的参数定义一个默认值：

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### 选项

选项和参数一样，它们也是用户的一种输入。但是它们在命令行被指定时会使用 `--` 作为前缀。这里有两种类型的选项：一种接收数值，而另一种不接收。对于不接收数值的选项，它们的功能就好像开关一样。让我们来看一下这种类型选项的示例：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在这个示例中，`--queue` 开关选项可以在使用 Artisan 命令时被指定。如果指定了 `--queque`，那么这个选项的值将会为 `true`。否则选项的值将为 `false`：

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### 附带值的选项

下面，让我们来看一下可以接收值的选项。你可以通过在选项后面添加 `=` 号来表明这个选项是需要通过用户输入的：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在这个例子中，用户可以传递一个值到选项，就像这样：

    php artisan email:send 1 --queue=default

你也可以分配一个默认值到选项，当选项值没有被用户指定时，那么它将会使用默认值：

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### 选项简写

你也可以为选项定义一个简写，你需要使用 `|` 来分割简写与选项名称，并将简写像下面的示例一样前置：

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### 输入数组

如果你想要定义参数或选项期望得到的是输入数组，你可以使用 `*` 通配符，首先，让我们来看一下指定数组参数的示例：

    email:send {user*}

当调用这个方法时，`user` 参数列可以按序的传递到命令行中。比如，下面的命令将会设置 `user` 的值为 `['foo', 'bar']`:

    php artisan email:send foo bar

当定义一个期待输入输入的选项时，你需要在每个选项值之前都添加选项名前缀：

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### 输入描述

你也可以分配描述信息到选项或者参数中，你需要使用 `:` 来进行分割描述和选项或参数，如果你需要一些额外的空间来定义你的命令描述，那么你可以自由的将它们格式化为多行：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## 命令 I/O

<a name="retrieving-input"></a>
### 检索输入

当你的命令工具存在时，你明显会需要访问命令行中所期望得到的参数或选项的值。你可以使用 `argument` 和 `option` 方法来得到：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

你可以通过使用 `arguments` 方法来获取所有参数所组成的数组：

    $arguments = $this->arguments();

选项可以非常简单的类似参数一样的通过 `option` 方法进行检索。你也可以使用 `options` 方法来返回所有选项所组成的数组：

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

如果相应的参数或选项没有被检索到，会返回 `null`。

<a name="prompting-for-input"></a>
### 为输入进行提示

除了显示输出之外，你可能也需要在执行你的命令行工具的过程中向用户索求额外的输入。你可以使用 `ask` 方法来进行提示用户输入，这个方法将接收用户的输入并返回输入的内容：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret` 方法与 `ask` 方法非常相似，只是用户在控制台中输入的内容并不是可见的。这个方法通常是询问用户密码相关时使用的：

    $password = $this->secret('What is the password?');

#### 要求确认

如果你只是需要用户的确认，你可以使用 `confirm` 方法。默认的，该方法返回 `false`. 但是如果用户在响应中输入了 `y`，该方法将返回 `true`.

    if ($this->confirm('Do you wish to continue? [y|N]')) {
        //
    }

#### 自动补全

`anticipate` 方法可以用来对可选的内容进行自动完成的提示。这里只是对用户有可能选择的内容进行自动完成提示，并非强制要求用户仅选择可选的内容:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### 多项选择

如果你需要给用户预置选项，你可以使用 `choice` 方法。用户必须选中选项中的索引，用户选中相应的索引的答案的值将会被返回。你可以设置一个默认的索引值，这个索引值将在用户没有做出任何选择时返回：

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $default);

<a name="writing-output"></a>
### 编写输出

你可以使用 `line`，`info`，`comment`，`question` 和 `error` 方法发送输出到控制台。这些方法会使用相应的 ANSI 颜色来表明相应的目的。比如，你可以使用 `info` 方法来向用户显示一个信息消息。通常，这条消息在控制台中是一个绿色的文本：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

你可以使用 `error` 方法来显示一个错误消息。错误消息通常都是红色的：

    $this->error('Something went wrong!');

你可以使用 `line` 方法来显示一个原生的控制台输出。`line` 方法并没有对消息设置任何的独特颜色信息：

    $this->line('Display this on the screen');

#### 表格布局

你可以使用 `table` 方法来简单的对多行或多列的数据进行格式化布局。你只需要向方法中传递头部和行信息到方法中就可以了。宽度和高度将会自动的通过所给定的数据进行计算：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 进度提示

对于一些耗时的任务来说，有一个进度提示是非常有用的。如果使用输出对象，我们就可以开始，推进和停止进度条。你需要在你开始进度之前定义步长。然后根据进行的每一步来推进进度条：

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

你可以通过查看 [Symfony Progress Bar component documentation](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html) 来获取更多的选项信息。

<a name="registering-commands"></a>
## 注册命令

一旦你完成了命令行的编写，你还需要注册其在 Artisan 命令中可用。这些需要在 `app/Console/Kernel.php` 文件中完成。在这个文件中，你会发现 `commands` 属性，它是一个命令行类的列表，你需要将命令类注册到这里。当 Artisan 启动时，所有在这个列表中的命令都会通过 [服务容器](/{{language}}/{{version}}/container) 解析到 Arisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programatically-executing-commands"></a>
## 通过代码执行命令

有时候你可能希望在控制台之外执行 Artisan 命令。比如，你希望在控制器或路由中触发 Artisan 命令。你可以使用 `Artisan` 假面的 `call` 方法来完成这些。`call` 方法接收一个命令名称，和一个包含所有参数和选项所组成的数组，命令执行完成之后会返回一个退出代码：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

通过使用 `Artisan` 假面的 `queue` 方法，你甚至可以队列化 `Artisan` 命令，在后台进程中 [队列工人](/{{language}}/{{version}}/queues) 会按序的帮你执行完成命令。在使用这个方法之前，请确保你已经配置好了你的队列并且运行了一个队列监听器：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

如果你需要强制指定一个不接受字符串值的选项的值为一个字符串，就像 `migrate:refresh` 命令中的 `--force` 选项，你可以明确的传递 `true` 或者 `false`：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### 从其它命令中调用命令

有时候，你可能希望在命令行工具中调用另外一个已经存在的 Artisan 命令。你可以使用 `call` 方法来完成这些。`call` 方法接收命令的名称和一个包含所有参数和选项的数组：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

如果你希望调用另外一个控制台命令而不希望它有任何的输出，你可以使用 `callSilent` 方法，`callSilent` 方法具有 `call` 方法相同的调用方式：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
