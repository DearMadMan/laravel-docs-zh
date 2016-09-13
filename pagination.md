# 分页

- [前言](#introduction)
- [基础用法](#basic-usage)
    - [对查询构造器结果进行分页](#paginating-query-builder-results)
    - [对 Eloquent 结果分页](#paginating-eloquent-results)
    - [手动的创建分页器](#manually-creating-a-paginator)
- [展示分页结果](#displaying-pagination-results)
    - [转换结果到 JSON](#converting-results-to-json)
- [自定义分页视图](#customizing-the-pagination-view)
- [分页器实例方法](#paginator-instance-methods)

<a name="introduction"></a>
## 前言

在其他框架中，分页通常是比较痛苦的。Laravel 的分页器集成了 [查询生成器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent)，并且为数据库结果提供了方便易用的分页。并且其生成的 HTML 是兼容 [Bootstrap CSS framework](http://getbootstrap.com/) 的。

<a name="basic-usage"></a>
## 基础用法

<a name="paginating-query-builder-results"></a>
### 对查询构建器结果进行分页

这里有几种方式来对元素进行分页。而最简单的方式就是通过使用 [查询生成器](/docs/{{version}}/queries) or [Eloquent 查询](/docs/{{version}}/eloquent) 的 `paginate` 方法。`paginate` 方法会根据当前用户所访问的当前页面来自动的设置正确的位移和显示的范围。默认的，当前页面是通过 HTTP 请求的查询字符串 `?page` 来自动获取的。当然，Laravel 会自动的检测到这个值，并且会自动的在生成的分页器的链接中插入合适的值。

首先，让我们来看一下在查询中调用 `paginate` 方法。在这个例子中，你仅需要在 `paginate` 方法中传递每页所需要展示的数量。让我们来设置每页展示的数量为 `15` :

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show all of the users for the application.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

            return view('user.index', ['users' => $users]);
        }
    }

> {note} 目前，Laravel 中使用 `groupBy` 语法不能正确的进行分页操作。如果你需要使用 `groupBy` 来对结果进行分页，推荐你手动的执行查询和进行分页。

#### "简单分页"

如果你只需要简单的显示 "上一页" 和 "下一页" 链接，你可以选择使用 `simplePaginate` 方法来执行一个有效的查询。这对于一些大型的数据库在视图中不需要显示每一页的链接时非常有效：

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### 对 Eloquent 结果进行分页

你也可以对 [Eloquent](/docs/{{version}}/eloquent) 查询的结果进行分页。我们来对 `User` 模型以每页显示 `15` 项的方式来进行分页。其语法和使用查询构造器进行分页非常像：

    $users = App\User::paginate(15);

当然，你也可以优先进行其他查询之后再进行分页操作：

    $users = User::where('votes', '>', 100)->paginate(15);

你同样也可以在使用 Eloquent 模型分页时使用 `simplePaginate` 方法：

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### 手动的创造分页器

有时候你可能需要手动的创建一个分页实例，它传递项目的数组。你可以创建 `Illuminate\Pagination\Paginator` 或者 `Illuminate\Pagination\LengthAwarePaginator` 实例，这取决于你的需求。

`Paginator` 类不需要知道结果中项目的总数量。事实上，正是因为这样，这个类并没有提供获取最后一页的方法。`LengthAwarePaginator` 类似于 `Paginator` 接收几乎同样的参数。但是，它需要在结果中包含项目的总数目。

另一方面，`Paginator` 对应查询构造器和 Eloquent 中的 `simplePaginate` 方法。而 `LengthAwarePaginator` 对应 `paginate` 方法。

> {note} 当手动的创建分页器实例时，你应该手动的对传递到分页器中的结果进行切片操作，如果你并不明白如何去进行分片，请查看 PHP 的 [array_slice](http://php.net/manual/en/function.array-slice.php) 方法。

<a name="displaying-pagination-results"></a>
## 展示分页结果

当你对查询构造器或者 Eloquent 查询使用 `paginate` 或者 `simplePaginate` 方法时，你可以获取到分页器的实例。当调用 `paginate` 方法时，你会获取到一个 `Illuminate\Pagination\LengthAwarePaginator` 实例，当调用 `simplePaginate` 方法时，你会获得一个 `Illuminate\Pagination\Paginator` 的实例。这些对象会提供多种方法来描述结果集。除了这些帮助方法，分页器本身也是一个迭代器，它可以像数组一样被循环操作。所以，一旦你获取到了分页结果，你可以像这样在 [Blade](/docs/{{version}}/blade) 视图中来显示和生成分页链接：

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

`links` 方法会根据结果集来生成分页结果。对每一个链接都会自动包含 `page` 查询变量。使用 `links` 方法生成的 HTML 是兼容 [Bootstrap CSS framework](https://getbootstrap.com) 框架的。

#### 自定义分页器 URL

`setPath` 方法允许你对分页器所生成的链接 URI 进行定制化。比如，你希望分页器生成类似于 `http://example.com/custom/url?page=N`，你可以传递 `custom/url` 到 `setPath` 方法：

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->setPath('custom/url');

        //
    });

#### 追加到分页连接

你可以通过使用 `appends` 方法来添加查询字符串到分页器链接中。比如，在分页器链接中追加 `sort=votes` 查询字符串。你可以像下面一样调用 `appends` 方法：

    {{ $users->appends(['sort' => 'votes'])->links() }}

如果你需要在分页器的 URLs 中添加一个 hash 分段，你可以使用 `fragment` 方法。比如，在分页链接中添加 `#foo`：

    {{ $users->fragment('foo')->links() }}

<a name="converting-results-to-json"></a>
## 转换结果为 JSON

Laravel 的分页器类实现了 `Illuminate\Contracts\Support\Jsonable` 契约并且暴露了 `toJson` 方法，所以它可以非常简单的将分页结果转换为 JSON 格式。你也可以在路由或者控制器动作中简单的返回分页器实例来进行 JSON 格式的响应：

    Route::get('users', function () {
        return App\User::paginate();
    });

分页器的 JSON 信息会包含许多的元数据，比如 `total`，`current_page`，`last_page` 等等。而实际的结果对象集会保存在 JSON 数组的 `data` 键中。下面是一个从路由中返回分页器实例的响应示例：

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "from": 1,
       "to": 15,
       "data":[
            {
                // Result Object
            },
            {
                // Result Object
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## 自定义分页视图

默认的，渲染后的分页连接视图是基于 Bootstrap CSS framework 的。事实上，如果你并没有使用 Bootstrap，你也可以自由的定义自己的视图来渲染这些分页连接。当调用一个分页器实例的 `links` 方法时，你可以传递一个视图的名称到 `links` 方法中：

    {{ $paginator->links('view.name') }}

事实上，自定义分页视图最简的的方式是通过使用 `vendor:publish` 命令将分页视图发布到 `resources/views/vendor` 目录中，然后进行按需的修改：

    php artisan vendor:publish --tag=laravel-pagination

这个命令会将视图放置到 `resources/views/vendor/pagination` 目录中。目录中的 `default.blade.php` 文件就是默认的分页视图。你可以简单的修改这个文件来改变分页的 HTML。

<a name="paginator-instance-methods"></a>
## 分页器实例方法

你可以通过调用分页器实例的方法来访问额外的分页信息：

- `$results->count()`
- `$results->currentPage()`
- `$results->firstItem()`
- `$results->hasMorePages()`
- `$results->lastItem()`
- `$results->lastPage() (Not available when using simplePaginate)`
- `$results->nextPageUrl()`
- `$results->perPage()`
- `$results->previousPageUrl()`
- `$results->total() (Not available when using simplePaginate)`
- `$results->url($page)`
