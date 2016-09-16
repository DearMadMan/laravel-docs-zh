# Laravel Scout

- [前言](#introduction)
- [安装](#installation)
    - [队列](#queueing)
    - [驱动需求](#driver-prerequisites)
- [配置](#configuration)
    - [配置模型索引](#configuring-model-indexes)
    - [配置可检索数据](#configuring-searchable-data)
- [索引](#indexing)
    - [批量导入](#batch-import)
    - [添加记录](#adding-records)
    - [更新记录](#updating-records)
    - [删除记录](#removing-records)
    - [暂停索引](#pausing-indexing)
- [检索](#searching)
    - [Where 子句](#where-clauses)
    - [分页](#pagination)
- [自定义引擎](#custom-engines)

<a name="introduction"></a>
## 前言

Laravel Scout 提供简单的基于驱动的解决方案来为你的  [Eloquent 模型](/{{language}}/{{version}}/eloquent) 添加全文检索的功能。使用模型观察者，Scout 可以自动的依据你的 Eloquent 记录来保持搜索索引的同步。

目前，Scout 自带支持 [Algolia](https://www.algolia.com/) 的驱动。事实上，你可以编写一个自定义驱动，并且你可以自由的继承 Scout 来做自己的搜索实现。

<a name="installation"></a>
## 安装

首先，你需要通过 Composer 包管理器来安装 Scout:

    composer require laravel/scout

然后，你应该添加 `ScoutServiceProvider` 到你的 `config/app.php` 配置文件中的 `providers` 数组中：

    Laravel\Scout\ScoutServiceProvider::class,

在你注册完 Scout 的服务提供者之后，你应该使用 `vendor:publish` Artisan 命令发布 Scout 的配置。这个命令会发布 `scout.php` 配置文件到你的 `config` 目录：

    php artisan vendor:publish

最后，你需要在需要支持全文检索的模型中添加 `Laravel\Scout\Searchable` trait。这个性状将会注册一个模型观察者来维持搜索驱动和模型之间数据的同步：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### 队列

虽然严格的来说，Scout 并不是强依赖于队列的。但是在你使用 Scout 之前还是推荐你配置 [队列驱动](/{{language}}/{{version}}/queues)。使用队列工人可以使 Scout 将模型信息同步到搜索索引的所有操作都可以队列化进行，这可以为应用的 web 交互接口提供更快速的响应。

当你配置完队列之后，你需要设置 `config/scout.php` 配置文件的 `queue` 选项为 `true`：

    'queue' => true,

<a name="driver-prerequisites"></a>
### 驱动依赖

#### Algolia

当使用 Algolia 驱动时，你应该在你的 `config/scout.php` 配置文件中配置你的 Algolia `id` 和 `secret` 凭证信息。当你配置完成这些之后，你还需要通过 Composer 来安装 Algolia PHP SDK:

    composer require algolia/algoliasearch-client-php

<a name="configuration"></a>
## 配置

<a name="configuring-model-indexes"></a>
### 配置模型索引

每一个 Eloquent 模型都会与一个搜索索引同步，这个索引会为模型提供所有可搜索的记录信息。换句话说，你可以认为这个索引就类似于 MySQL 中的表。默认的，每个模型都以典型的表名来持久化到索引中。通常，这些表名都是模型名称的复数形式。但是，你可以通过复写模型中的 `searchableAs` 方法来自定义这个模型的索引：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the index name for the model.
         *
         * @return string
         */
        public function searchableAs()
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### 配置可检索的数据

默认的，模型会以整个 `toArray` 的格式化的形式持久化到搜索索引中。如果你希望自定义同步到搜索索引中的数据，那么你可以复写模型中的 `toSearchableArray` 方法：

    <?php

    namespace App;

    use Laravel\Scout\Searchable;
    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        use Searchable;

        /**
         * Get the indexable data array for the model.
         *
         * @return array
         */
        public function toSearchableArray()
        {
            $array = $this->toArray();

            // Customize array...

            return $array;
        }
    }

<a name="indexing"></a>
## 索引

<a name="batch-import"></a>
### 批量导入

如果你在一个已存在的项目中安装 Scout，那么你可能会希望将数据库中的记录导入到你的搜索引擎中。Scout 提供了 `import` Artisan 命令来帮助你将所有已经存在的记录导入到搜索索引中：

    php artisan scout:import "App\Post"

<a name="adding-records"></a>
### 添加记录

当你为模型添加 `Laravel\Scout\Searchable` trait 之后，你所需要做的就是 `save` 一个模型的实例，它会自动的被添加到搜索索引中去。如果你为 Scout 配置了 [队列](#queueing)，那么这个操作将会在后台递交给队列工人去执行：

    $order = new App\Order;

    // ...

    $order->save();

#### 通过查询添加

如果你想通过 Eloquent 查询来为一个模型集合添加搜索索引。那么你可以在 Eloquent 查询中链式的调用 `searchable` 方法。`searchable` 方法将会 [将查询结果分块](/{{language}}/{{version}}/eloquent#chunking-results)，并且会将这些记录添加到搜索索引中。这次，如果你为 Scout 配置了 [队列](#queueing)，那么这个操作将会在后台递交给队列工人去执行：

    // Adding via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also add records via relationships...
    $user->orders()->searchable();

    // You may also add records via collections...
    $orders->searchable();

`searchable` 方法可以被认为是一个 "插入" 操作。换句话说，如果你的索引中已经存在了模型的记录，那么它将会被更新。如果没有，那么它将会被添加到索引中。

<a name="updating-records"></a>
### 更新记录

为了更新可检索模型的索引，你只需要更新模型实例的属性，然后使用 `save` 方法将它更新到数据库。Scout 将会自动的持久化这个变化到你的搜索索引中：

    $order = App\Order::find(1);

    // Update the order...

    $order->save();

你也可以在 Eloquent 查询中使用 `searchabale` 方法来为模型的集合更新索引。如果你的搜索索引中不存在这些模型，那么它们将会被创建：

    // Updating via Eloquent query...
    App\Order::where('price', '>', 100)->searchable();

    // You may also update via relationships...
    $user->orders()->searchable();

    // You may also update via collections...
    $orders->searchable();

<a name="removing-records"></a>
### 删除记录

如果你想要移除你的索引，那么你只需要简单的调用 `delete` 方法将模型从数据库中删除就可以了。这种删除的形式甚至可以兼容 [软删除](/{{language}}/{{version}}/eloquent#soft-deleting) 的模型：

    $order = App\Order::find(1);

    $order->delete();

如果你不想某些模型被检索到，那么你可以在 Eloquent 查询实例或集合中使用 `unsearchable` 方法：

    // Removing via Eloquent query...
    App\Order::where('price', '>', 100)->unsearchable();

    // You may also remove via relationships...
    $user->orders()->unsearchable();

    // You may also remove via collections...
    $orders->unsearchable();

<a name="pausing-indexing"></a>
### 暂停索引

有时候你或许想要对一些模型进行批量的处理操作，但是却不想这些操作同步到搜索索引中去。那么你可以使用 `withoutSyncingToSearch` 方法。这个方法接收一个单独的回调作为参数，它会被立即执行。任何在回调中所进行的模型操作都不会被同步到模型的索引中：

    App\Order::withoutSyncingToSearch(function () {
        // Perform model actions...
    });

<a name="searching"></a>
## 检索

你可以使用 `search` 方法来进行模型的检索。检索方法接收一个单独的字符串作为参数来检索你的模型。你应该继续的链式的调用 `get` 方法来获取那些匹配的检索查询结果：

    $orders = App\Order::search('Star Trek')->get();

由于 Scout 的检索结果会返回 Eloquent 模型的集合，所以你可以直接在路由或控制器中返回结果，它们将会被自动的转换为 JSON：

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

<a name="where-clauses"></a>
### Where 子句

Scout 允许你添加简单的 "where" 子句到你的搜索查询。目前，这些子句只支持基础的数字等值匹配，并且它们常用语通过目标 ID 来限制搜索查询的结果。由于搜索索引并不是一个关联数据库，所以一些更高级的 "where" 子句并没有被支持：

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### 分页

除了可以检索模型的集合之外，你也可以使用 `paginate` 方法来对结果进行分页。这个方法会返回一个 `Paginator` 实例，就和 [传统的 Eloquent 查询分页](/{{language}}/{{version}}/pagination) 一样：

    $orders = App\Order::search('Star Trek')->paginate();

你可以通过指定一个数值参数到 `paginate` 方法来表明每页检索的模型数量：

    $orders = App\Order::search('Star Trek')->paginate(15);

当你获得检索的结果之后，你也可以像传统的 Eloquent 查询分页一样使用 [Blade](/{{language}}/{{version}}/blade) 为结果提供展示和选择分页链接：

    <div class="container">
        @foreach ($orders as $order)
            {{ $order->price }}
        @endforeach
    </div>

    {{ $orders->links() }}

<a name="custom-engines"></a>
## 自定义引擎

#### 编写引擎

如果內建的 Scout 搜索引擎并不是你所需要的，那么你可以编写你自己的自定义引擎同 Scout 一起注册。你的引擎应该继承 `Laravel\Scout\Engines\Engine`抽象类。这个抽象类包含了 5 个方法，这些方法需要你的引擎去进行实现：

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function map($results, $model);

你可以通过浏览 `Laravel\Scout\Engines\AlgoliaEngine` 类中的实现来更快的了解如何实现自己的引擎。

#### 注册引擎

当你编写完成自定义引擎之后，你可以使用 Scout 引擎管理者的 `extend` 方法来注册它。你应该在你的 `AppServiceProvider` 或者其他服务提供者的 `boot` 方法中进行 `extend` 方法的调用。比如，如果你编写了 `MySqlSearchEngine`，那么你可以这样进行注册：

    use Laravel\Scout\EngineManager;

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

当你完成引擎的注册之后，你需要在 `config/scout.php` 配置文件中将默认的 Scout `driver` 指定为你所注册的引擎：

    'driver' => 'mysql',
