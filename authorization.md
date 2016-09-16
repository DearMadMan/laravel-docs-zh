# 授权

- [前言](#introduction)
- [Gates](#gates)
    - [编写 Gates](#writing-gates)
    - [授权动作](#authorizing-actions-via-gates)
- [创建策略](#creating-policies)
    - [生成策略](#generating-policies)
    - [注册策略](#registering-policies)
- [编写策略](#writing-policies)
    - [策略方法](#policy-methods)
    - [模型之外](#methods-without-models)
    - [策略过滤](#policy-filters)
- [使用策略进行授权](#authorizing-actions-using-policies)
    - [通过用户模型](#via-the-user-model)
    - [通过中间件](#via-middleware)
    - [通过控制器辅助函数](#via-controller-helpers)
    - [通过 Blade 模板](#via-blade-templates)

<a name="introduction"></a>
## 前言

Laravel 除了提供开箱即用的 [认证服务](/{{language}}/{{version}}/authentication)，还提供了许多简单的方式来管理授权逻辑和资源的访问控制。就像认证一样，Laravel 的授权方法非常简单，它主要通过两种形式作出授权动作：gates 和 策略

你可以将 gates 和 策略想象为路由和控制器。Gates 提供了基于闭包的方法来进行授权操作，而策略就像控制器一样，将特定的模型或资源组织在一起。我们接下来将会先剖析 gates，然后再来探讨一下策略。

需要特别指出的是在应用中，gates 和策略并不是互相排斥的。在大多数应用中都有可能会混合的使用 gates 和策略，没关系，这都可以很好的协作。Gates 多适用于与模型或资源无关的授权动作，比如浏览管理员控制面板，而策略常用于为特定的模型或者资源进行授权动作。

<a name="gates"></a>
## Gates（能力）

<a name="writing-gates"></a>
### 编写 Gates

Gates 是一些闭包函数，它们用于判断用户对指定的动作是否有执行的权利，通常它们应该是使用 `Gate` 假面被定义在 `App\Providers\AuthServiceProvider` 类中。Gates 总是接收用户实例作为第一个参数，你也可以传递一些额外可选的参数，比如相关的 Eloquent 模型：

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', function ($user, $post) {
            return $user->id == $post->user_id;
        });
    }

<a name="authorizing-actions-via-gates"></a>
### 授权动作

如果要使用 gates 对一个动作进行授权，那么你应该使用 `allows` 方法。你需要注意的是，你并不需要传递当前已经被认证的用户到 `allows` 方法中，Laravel 会自动的将其传递到 gate 的闭包中:

    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    });

如果你想要判断特定的用户是否有执行某个动作的权限，那么你可以使用 `Gate` 假面的 `forUser` 方法:

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // The user can update the post...
    }

<a name="creating-policies"></a>
## 创建策略

<a name="generating-policies"></a>
### 生成策略

策略就是一些类文件，它们用来管理围绕特定模型或资源的授权逻辑。比如，如果你的应用是一个博客，那么你可以拥有一个 `Post` 模型和相应的 `PostPolicy` 来对如创建或者更新 post 的用户动作进行授权。

你可以通过 `make:policy` [artisan command](/{{language}}/{{version}}/artisan) 命令来生成一个策略。所生成的策略会存放在 `app/Policies` 目录，如果该目录不存在，那么 Laravel 将会自动的为你创建：

    php artisan make:policy PostPolicy

`make:policy` 命令将会生成一个空的策略类。如果你希望直接生成的类文件中已经包含了基础的 CRUD 策略方法，那么你可以在执行该命令时指定 `--model` 选项:

    php artisan make:policy PostPolicy --model=Post

> {tip} 所有的策略类都是通过 [服务容器](/{{language}}/{{version}}/container) 解析而来。这意味着你可以使用类型提示来在策略类的构造函数中进行依赖注入所需要的依赖。

<a name="registering-policies"></a>
### 注册策略

一旦策略存在，我们还需要在对其进行注册。在 `AuthServiceProvider` 中包含了一个 `policies` 属性，该属性存放所有实体与策略间的映射。注册这些策略可以指导 Laravel 对给定模型的授权动作使用相应的策略: 

    <?php

    namespace App\Providers;

    use App\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * The policy mappings for the application.
         *
         * @var array
         */
        protected $policies = [
            Post::class => PostPolicy::class,
        ];

        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            //
        }
    }

<a name="writing-policies"></a>
## 编写策略

<a name="policy-methods"></a>
### 策略方法

一旦策略被生成和注册后，我们就可以为所有动作的授权添加验证方法。例如，让我们在 `PostPolicy` 类中定义一个 `update` 方法，用来验证所给定的用户是否具有 `update` 给定 `Post` 实例的能力：

`update` 方法会接收一个 `User` 和一个 `Post` 实例作为它的参数，它应该明确的返回 `true` 或者 `false` 来决定用户是否有更新给定 `Post` 的权利。那么，作为这个示例，让我们来验证一下用户的 `id` 是否与 post 的 `user_id` 相匹配:

    <?php

    namespace App\Policies;

    use App\User;
    use App\Post;

    class PostPolicy
    {
        /**
         * Determine if the given post can be updated by the user.
         *
         * @param  \App\User  $user
         * @param  \App\Post  $post
         * @return bool
         */
        public function update(User $user, Post $post)
        {
            return $user->id === $post->user_id;
        }
    }

你可以继续在策略中添加用于验证其它动作是否授权的方法。比如，你可以定义一个 `view` 或者 `delete` 方法去对 `Post` 的各种动作进行授权，但是你应该记住，你可以自由的定义这些策略方法的名称。

> {tip} 如果你在执行 Artisan 命令时使用了 `--model` 选项来生成策略，那么它已经为 `view`，`create`，`update`，和 `delete` 动作生成了相应的方法。

<a name="methods-without-models"></a>
### 模型之外

一些策略仅仅接收已经被认证的用户作为参数，而且并不包含当前所授权模型的实例。这种场景多用于对 `create` 动作进行授权，比如，如果你在创建一个博客，你可能会检查用户是否已经被授权可以创建任意的 posts。

当定义策略方法而不接收模型的实例时，就像 `create` 方法那样，它不会接收模型的实例作为参数，那么你所定义的方法应该仅接收一个已被认证的用户作为参数:

    /**
     * Determine if the given user can create posts.
     *
     * @param  \App\User  $user
     * @return bool
     */
    public function create(User $user)
    {
        //
    }

> {tip} 如果你在执行 Artisan 命令时使用了 `--model` 选项来生成策略，那么在所生成的策略类中已经包含了所有关联 CRUD 的策略方法。

<a name="policy-filters"></a>
### 策略过滤

对于一些用户，你可能希望在给定的策略中赋予它们所有动作的授权，那么你可以在策略中定义一个 `before` 方法，这给予你在给定的策略方法被执行之前为动作进行授权的机会。这个特性常用来为应用的管理者提供任意动作的执行权限：

    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }

<a name="authorizing-actions-using-policies"></a>
## 使用策略对动作进行授权

<a name="via-the-user-model"></a>
### 通过用户模型

你可以通过 `User` 模型的实例来检查用户的能力。默认的 Laravel 的 `App\User` 模型使用了 `Authorizable` trait，这个性状包含两个方法：`can` 和 `cannot`。这两个方法和 `Gate` 假面的 `allows` 和 `denies` 方法的用法相同。我们还使用上面曾使用过的例子，修改成如下：

Laravel 应用自带的 `User` 模型包含了两个有用的方法来对动作进行授权：`can` 和 `cant`。`can` 方法接收你希望授权的动作和所关联的模型。比如，让我们来判断用户是否具有对给定的 `Post` 模型的 'update' 动作的授权:

    if ($user->can('update', $post)) {
        //
    }

如果 [策略](#registering-policies) 是为给定的模型而注册的。那么 `can` 方法会自动的调用适当的策略，并且会返回布尔结果。如果没有为给定的模型注册相应的策略，那么 `can` 方法会尝试调用基于 Gate 所匹配的给定的动作名称中的闭包方法。

#### 不依赖于模型的动作

还记得吗，一些像 `create` 的动作并不需要引入关联模型的实例。在这种场景下，你可以直接传递一个类的名称到 `can` 方法中。这个类名称会被用来判断在对动作进行授权时应使用哪种策略。

    use App\Post;

    if ($user->can('create', Post::class)) {
        // Executes the "create" method on the relevant policy...
    }

<a name="via-middleware"></a>
### 通过中间件

Laravel 引入的中间可以在进入的请求到达路由或者控制器之前对动作进行授权。默认的 `Illuminate\Auth\Middleware\Authorize` 中间件在你的 `App\Http\Kernel` 类中被分配了 `can` 作为索引键。那么，让我们来看一下使用 `can` 中间件来对用户是否具有更新博客 post 的授权的示例吧:

    use App\Post;

    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');

在这个示例中，我们为 `can` 中间件传递了两个参数。第一个参数是我们想要进行授权验证的动作名称，第二个参数则是我们需要传递到策略方法中的路由参数名称。上述例子中，由于我们是使用的 [隐式模型绑定](/{{language}}/{{version}}/routing#implicit-binding)，所以一个 `Post` 模型的实例将会传递到策略方法中。如果用户并没有被授权访问跟多的动作，那么中间件会自动的生成一个 `403` 状态码的 HTTP 响应。

#### 不依赖于模型的动作

再一次，一些像 `create` 的动作并不需要引入关联模型的实例。在这种场景下，你可以直接传递一个类的名称到中间件中。这个类名称会被用来判断在对动作进行授权时应使用哪种策略。

    Route::post('/post', function () {
        // The current user may create posts...
    })->middleware('can:create,App\Post');

<a name="via-controller-helpers"></a>
### 通过控制器辅助方法

除了为 `User` 模型提供了一些有用的方法外，Laravel 也对于继承自 `App\Http\Controllers\Controller` 类的控制器提供了 `authorize` 方法。就像 `can` 方法一样，这个方法接收你所希望进行授权的动作的名称和所关联的模型。如果动作没有被授权，那么 `authorize` 方法将会抛出 `Illumiante\Auth\Access\AuthorizationException`，这在默认的 Laravel 异常处理器中会被转换为 `403` 状态码的 HTTP 响应：

    <?php

    namespace App\Http\Controllers;

    use App\Post;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Update the given blog post.
         *
         * @param  Request  $request
         * @param  Post  $post
         * @return Response
         */
        public function update(Request $request, Post $post)
        {
            $this->authorize('update', $post);

            // The current user can update the blog post...
        }
    }

#### 不依赖于模型的动作

就如我们上面所提到的，一些像 `create` 的动作并不需要引入关联模型的实例。在这种场景下，你可以直接传递一个类的名称到 `authorize` 方法中。这个类名称会被用来判断在对动作进行授权时应使用哪种策略。

    /**
     * Create a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // The current user can create blog posts...
    }

<a name="via-blade-templates"></a>
### 通过 Blade 模板

在编写 Blade 模板时，你或许希望只有在用户具有给定动作的授权时才显示页面中的一部分。比如，你或许希望只有在用户可以更新文章时，才展示更新相关的表单。在这种场景下，你可以使用 `@can` 和 `@cannot` 指令。

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @endcan

    @cannot('update', $post)
        <!-- The Current User Can't Update The Post -->
    @endcannot

这些指令是专门为了方便简化这种场景下的 `@if` 和 `@unless` 声明而存在的。上面使用 `@can` 和 `@cannot` 指令所处理的逻辑也可以转换为下面的声明

    @if (Auth::user()->can('update', $post))
        <!-- The Current User Can Update The Post -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- The Current User Can't Update The Post -->
    @endunless

#### 不依赖于模型的动作

就像其它的授权方法那样，对于不依赖于模型的动作，你可以传递类的名称到 `@can` 和 `@cannot` 指令中：

    @can('create', Post::class)
        <!-- The Current User Can Create Posts -->
    @endcan

    @cannot('create', Post::class)
        <!-- The Current User Can't Create Posts -->
    @endcannot
