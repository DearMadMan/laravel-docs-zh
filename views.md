# 视图

- [基础用法](#basic-usage)
    - [传递数据到视图](#passing-data-to-views)
    - [共享数据到所有视图](#sharing-data-with-all-views)
- [视图 Composers](#view-composers)

<a name="basic-usage"></a>
## 基础用法

视图为表现逻辑与应用逻辑的分离提供了便利，Laravel 中所有的视图都被存储在 `resources/views` 目录下。

一个简单的视图看起来是这样的：

    <!-- View stored in resources/views/greeting.php -->

    <html>
        <body>
            <h1>Hello, <?php echo $name; ?></h1>
        </body>
    </html>

由于这个视图被存储在 `resources/views/greeting.php` 文件中，所以我们可以使用全局帮助方法 `view` 来返回渲染的结果：

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

你可以看到，`view` 方法接收的第一个参数是相对于目录 `resource/views` 的文件名。该方法允许传递数组作为第二个参数，数组中的键值对会作为相应的变量传递到视图。上面的例子中，我们传递了 `name` 变量，该变量会在渲染时执行 `echo` 方法返回。

当然，视图也可以嵌套的存放在 `resources/views` 的子目录中。你可以使用 "." 语法来引用嵌套的视图，比如，如果你的视图被存放在 `resources/views/admin/profile.php` 文件中，你可以像下面这样进行引用：

    return view('admin.profile', $data);

#### 假定视图存在

如果你需要假定一个视图存在，你可以在调用无参数的 `view` 帮助方法之后链式调用 `exists` 方法。这个方法将会在视图存在时返回 `true`：

    if (view()->exists('emails.customer')) {
        //
    }

当调用 `view` 帮助方法而不带任何参数时，它将会返回一个 `Illuminate\Contracts\View\Factory` 的实例，这个实例允许你访问该契约的任意方法。

<a name="view-data"></a>
### View Data

<a name="passing-data-to-views"></a>
#### 传递参数到视图

就如你在前面的示例所看到的，你可以轻易的传递一个数组数据到视图中:

    return view('greetings', ['name' => 'Victoria']);

使用这种方法传递数据，`$data` 必须是键值对的形式，在视图中你可以使用相应的键来访问相应的变量，就像 `<?php echo $key; ?>`。另外，你也可以使用使用 `with` 方法来追加额外的数据到视图：

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### 共享数据到所有视图

有时候，你可能需要共享一部分数据到所有的视图中，你可以使用视图工厂的 `share` 方法，通常，你应该在服务容器的 `boot` 方法中去做数据共享操作。你可以在 `AppServiceProvider` 或者独立生成一个分离的服务提供者来做这件事：

    <?php

    namespace App\Providers;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            view()->share('key', 'value');
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

<a name="view-composers"></a>
## 视图 Composers

视图 composers 就是当视图被渲染时会被触发的回调方法或者类的方法。如果你想要视图在每次渲染时都自动的绑定一些数据到视图，那么视图 composer 就可以帮助你分离这些逻辑处理到一个单独的文件进行有效的管理。

那么现在让我们在 [服务提供者](/docs/{{language}}/{{version}}/providers) 中注册我们自己的视图 composers。我们将使用 `view` 帮助方法来返回一个 `Illuminate\Contracts\View\Factory` 实现的实例。请注意，Laravel 并没有提供一个默认的目录去管理这些视图 Composers，你可以自由的按照自己的喜好去管理这些。例如，你当然可以创建一个 `App\Http\ViewComposer` 目录：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            view()->composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            view()->composer('dashboard', function ($view) {
                //
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

请注意，如果你创建了一个新的服务提供者来管理你的视图 composer 的注册，你千万不要忘记在 `config/app.php` 文件中的 `providers` 数组里进行该提供者的注册。

现在我们已经有了已经注册了的 composer，方法 `ProfileComposer@composer` 将在视图 `profile` 每次被渲染时自动执行。接着，我们来定义 `ProfileComposer` 类:

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

每当视图被渲染之前，composer 的 `compose` 方法都会被调用，并且会被传递一个 `Illuminate\View\View` 实例。你可以使用 `with` 方法来添加额外的数据到视图。

> **注意** 所有的视图 composers 都是通过 [服务容器](/docs/{{language}}/{{version}}/container) 解析的，所以你可以在 composer 的构造函数中添加类型提示来进行任意的依赖注入。

#### 附加 Composer 到多个视图

你可以一次性的绑定一个 composer 到多个视图，你只需要简单的在 `composer` 方法中的第一个参数传递一个包含视图名称的数组：

    view()->composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

事实上 `composer` 方法接收 `*` 字符串作为通配符，允许你附加 composer 到所有的视图中:

    view()->composer('*', function ($view) {
        //
    });

### View 创造者

视图 **创造者** 和视图 composer 非常相似。但是它会在视图实例化后就立即触发注册的方法，而不是等到视图渲染时。你可以使用 `creator` 方法来注册视图创造者:

    view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
