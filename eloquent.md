# Eloquent: 起步

- [前言](#introduction)
- [定义模型](#defining-models)
    - [Eloquent 模型约定](#eloquent-model-conventions)
- [检索模型](#retrieving-models)
    - [集合](#collections)
    - [结果分块](#chunking-results)
- [检索单个模型 / 聚合](#retrieving-single-models)
    - [聚合检索](#retrieving-aggregates)
- [插入 & 更新模型](#inserting-and-updating-models)
    - [插入](#inserts)
    - [更新](#updates)
    - [集体赋值](#mass-assignment)
    - [其它创建方法](#other-creation-methods)
- [删除模型](#deleting-models)
    - [软删除](#soft-deleting)
    - [查询软删除的模型](#querying-soft-deleted-models)
- [区间查询](#query-scopes)
    - [全局区间](#global-scopes)
    - [当前区间](#local-scopes)
- [事件](#events)
    - [观察者](#observers)

<a name="introduction"></a>
## 前言

Laravel 的 Eloquent ORM 提供了一种漂亮简洁的关系映射模型来与数据库进行交互。所有的数据库表都有相应的模型，这些模型被用来与表进行交互。模型允许你直接查询数据库表中的数据，及插入新的记录到数据表中。

在开始之前，你需要确保完成了 `config/database.php` 配置文件中的数据库配置。对于更多的配置数据库相关的信息，请参考 [文档](/docs/{{version}}/database#configuration)。

<a name="defining-models"></a>
## 定义模型

在开始之前，让我们先来创建一个 Eloquent 模型。模型通常存放在 `app` 目录下，但是你也可以自由的放置在任何地方，只要它能够根据你的 `composer.json` 文件的指导进行自动的加载。所有的 Eloquent 模型都需要继承 `Illuminate\Database\Eloquent\Model` 类。

最简单的创建一个模型类的方式就是使用 `make:model` [Artisan 命令](/docs/{{version}}/artisan)：

    php artisan make:model User

如果你希望在你生成模型的同时生成一份 [database migration](/docs/{{version}}/migrations)，你可以使用 `--migration` 或者 `-m` 选项：

    php artisan make:model User --migration

    php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
### Eloquent 模型约定

现在，让我们来看一个 `Flight` 模型的示例，我们将通过它获取和存储数据库中 `flights` 表的数据：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        //
    }


#### 表名称

你需要注意我们上面的代码中并没有指出 `Flight` 模型使用哪个数据库的表。如果你没有明确的指出模型所对应的表，那么 Eloquent 将使用类的蛇形命名的复数形式来使用相应的数据表。所以，在这个例子中，Eloquent 将会假定 `Flight` 模型存储的记录被放在 `flights` 表中。你可以使用 `table` 属性来指定表名：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

#### 主键

Eloquent 也会假定所有表的主键列名为 `id`。你也可使用 `$primaryKey` 属性来手动的指定表的主键。

另外，Eloquent 也会假定主键是一个自增的整型值，这意味着，在默认情况下主键会被自动的转为 `int`。如果你希望使用非自增或者非数字的主键，那么你需要在模型中将公开的 `$incrementing` 属性设置为 `false`。

#### 时间戳

默认的，Eloquent 会认为模型表中存在 `created_at` 和 `updated_at` 列，如果你不希望 Eloquent 自主的管理这两列，你可以在模型中设置 `$timestamps` 属性为 `false`:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
    }

如果你需要自定义时间戳的格式，你可以设置 `$dateFormat` 属性。这个属性用来决定日期属性存储在数据库中的格式，以及模型在进行序列化为数组或者 JSON 时的显示样式：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

#### 数据库连接

默认的，所有的 Eloquent 模型都会使用应用配置的默认的数据库连接。如果你希望模型使用不同的数据库连接，你可以使用 `$connection` 属性：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The connection name for the model.
         *
         * @var string
         */
        protected $connection = 'connection-name';
    }

<a name="retrieving-models"></a>
## 检索多个模型

一旦你创建了模型和 [其相应的数据表](/docs/{{version}}/migrations#writing-migrations)，你就为从数据库中检索数据做好了准备。你可以把 Eloquent 模型作为一个强大的 [查询生成器](/docs/{{version}}/queries) 来使用，它允许通过模型流畅的查询相应的表中的数据。比如：

    <?php

    use App\Flight;

    $flights = App\Flight::all();

    foreach ($flights as $flight) {
        echo $flight->name;
    }

#### 添加额外的条件

Eloquent 的 `all` 方法会返回模型表中所有的记录。由于所有的 Eloquent 模型都可以作为 [查询生成器](/docs/{{version}}/queries) 来进行服务，所以你可以在这些查询中增加额外的条件，然后使用 `get` 方法来检索结果：

    $flights = App\Flight::where('active', 1)
                   ->orderBy('name', 'desc')
                   ->take(10)
                   ->get();

> {tip} 由于 Eloquent 模型也是查询生成器，所以你可以回顾一下 [查询生成器](/docs/{{version}}/queries) 中所有可用的方法，因为它们同样可以在 Eloquent 中进行使用。

<a name="collections"></a>
### 集合

对于像 `all` 和 `get` 这样的 Eloquent 方法会返回多个结果，这将返回一个 `Illuminate\Database\Eloquent\Collection` 实例。`Collection` 类提供了 [多种有用的方法](/docs/{{version}}/eloquent-collections#available-methods)与 Eloquent 结果进行交互:

    $flights = $flights->reject(function ($flight) {
        return $flight->cancelled;
    });

当然，你可以简单的对集合进行循环，因为他实现了 ArrayAccess 接口：

    foreach ($flights as $flight) {
        echo $flight->name;
    }

<a name="chunking-results"></a>
### 对结果分块

如果你需要处理数千条 Eloquent 记录，那么你可以使用 `chunk` 方法。`chunk` 方法会从 Eloquent 模型中检索出一小块记录，然后将其传递给 `Closure` 进行处理。使用 `chunk` 方法在处理大量结果集时可以有效的降低内存的消耗：

    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

传递到方法的第一个参数是你要设置进行分块的大小。而传递到第二个参数的闭包将会在从数据库检索出每块内容时进行调用。在闭包执行之前会进行当前块的数据库查询操作，紧接着，查询的结果将会传递到闭包中。

#### 使用指针

`cursor` 方法允许你使用指针的方式来遍历你的数据库记录，并且它只执行一条查询语句。当处理大量的数据时，`cursor` 方法可以极大的降低你的内存消耗：

    foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
        //
    }

<a name="retrieving-single-models"></a>
## 检索单个模型 / 聚合

当然，除了可以从表中检索出所有的记录之外，你也可以使用 `find` 和 `first` 方法来从数据库中检索出单条的数据。这将会返回一个单独的模型实例：

    // Retrieve a model by its primary key...
    $flight = App\Flight::find(1);

    // Retrieve the first model matching the query constraints...
    $flight = App\Flight::where('active', 1)->first();

你也可以使用 `find` 方法时传递一个包含主键所组成的数组，这将返回所有匹配到记录的集合：

    $flights = App\Flight::find([1, 2, 3]);

#### 未发现时的异常

有时候，你可能希望在没有找到匹配的模型时抛出一个异常。这对路由或者控制器来说尤其有用。`findOrFail` 和 `firstOrFail` 方法可以从查询中检索出首个匹配的结果，但是，如果结果中没有匹配的项，那么会抛出一个 `Illuminate\Database\Eloquent\ModelNotFoundException` 异常：

    $model = App\Flight::findOrFail(1);

    $model = App\Flight::where('legs', '>', 100)->firstOrFail();

如果异常没有被捕获，那么会直接传递给用户一个 `404` HTTP 响应。所以当你在使用这些方法时是没有必要编写明确的检查并返回 `404` 响应的：

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

<a name="retrieving-aggregates"></a>
### 聚合检索

当然，你可以使用 `count`，`sum`，`max` 和其他 [查询生成器](/docs/{{version}}/queries) 所提供的的 [聚合方法](/docs/{{version}}/queries#aggregates)。这些方法会返回适当的数值，而不是完整的模型实例：

    $count = App\Flight::where('active', 1)->count();

    $max = App\Flight::where('active', 1)->max('price');

<a name="inserting-and-updating-models"></a>
## 插入 & 更新模型

<a name="inserts"></a>
### 插入

想要在数据库中插入新的记录，只需要简单的创建模型实例，然后设置模型的属性，再调用 `save` 方法就可以了：

    <?php

    namespace App\Http\Controllers;

    use App\Flight;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class FlightController extends Controller
    {
        /**
         * Create a new flight instance.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate the request...

            $flight = new Flight;

            $flight->name = $request->name;

            $flight->save();
        }
    }

在这个例子中，我们简单的将流入的 HTTP 请求中的 `name` 参数分配到 `App\Flight` 模型实例的 `name` 属性上。当我们调用 `save` 方法时，这条记录将会插入到数据库中，同时 `created_at` 和 `updated_at` 时间戳也会自动的被进行设置，所以这并不需要进行手动的设置它们。

<a name="updates"></a>
### 更新

`save` 方法也可用来更新数据库中已经存在的数据模型。为了更新模型，你应该首先检索到它们，然后设置任何你想要更新的属性，接着调用 `save` 方法。这一次，`updated_at` 时间戳会自动的进行更新，所以你并不需要手动的更新这个值：

    $flight = App\Flight::find(1);

    $flight->name = 'New Flight Name';

    $flight->save();

#### 集体更新

你也可以针对查询匹配的多个模型进行更新操作。比如，所有 `active` 并且 `destination` 为 `San Diego` 的航班将会被标记为延迟：

    App\Flight::where('active', 1)
              ->where('destination', 'San Diego')
              ->update(['delayed' => 1]);

`update` 方法接收一个所要更新列的键值对所组成的数组。

> {note} 当通过 Eloquent 来发布集体更新时，被更新的模型并不会触发 `saved` 和 `updated` 模型事件。这是因为当发布集体更新时，被更新的模型事实上是从来没有被检索过的。

<a name="mass-assignment"></a>
### 集体赋值

你也可以使用 `create` 方法来在一行中存储一个新的模型。一个新增的模型实例将会从方法中返回。事实上，在开始做这些之前，你需要先指定模型的 `fillable` 或者 `guarded` 属性，它们是针对批量赋值时的保护措施。

当用户通过 HTTP 请求传递一些意外的参数时，这可能会造成批量赋值时一些问题的出现，或许参数中存在一些你并不想改变的列值的参数。比如，一个恶意的用户可能会在请求中嵌入一个 `is_admin` 参数，然后这些参数映射到你的 `create` 方法中，这就使用户将自己升级为了超级管理员。

所以，在开始之前，你需要先定义哪些模型属性是可以进行批量分配的。你可以使用模型 `$fillable` 属性，让我们在 `Flight` 模型中添加允许 `name` 属性的批量赋值：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = ['name'];
    }

一旦我们使一个属性可以进行批量赋值。那么我们可以使用 `create` 方法直接的添加记录到数据库中。`create` 方法会返回存储后的模型实例：

    $flight = App\Flight::create(['name' => 'Flight 10']);

#### 守卫属性

`$fillable` 就像是为批量赋值提供了一种白名单的机制。你也可以选择使用 `$guarded`。`$guarded` 属性应该包含一个你不想要进行批量赋值的属性所组成的数组。所有不在这个数组中的属性都将可以进行批量赋值。所以，`$guarded` 就像一个黑名单的功能。当然，你应该只使用 `$fillable` 或者 `$guarded` 中的一个，而不是全部。在下面的例子中，除了 `price`，所有的属性都可以进行批量赋值:

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The attributes that aren't mass assignable.
         *
         * @var array
         */
        protected $guarded = ['price'];
    }

如果你需要所有的属性都允许进行集体赋值，那么你可以将 `$guarded` 设置为空数组：

    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### 其它创建方法

这里也有其他两种方法可以使用批量赋值的方式进行模型的创建：`firstOrCreate` 和 `firstOrNew`。`firstOrCreate` 方法会尝试根据给定的列的键值对从数据库定位相关的模型。如果相关的模型没有找到，那么会根据给定的属性在数据库中新增一条记录。

`firstOrNew` 方法和 `firstOrCreate` 方法一样，但是它在模型没找到时只会返回一个新的模型实例，并不会在数据库中新增一条记录。你需要手动的使用 `save` 方法来将其存储到数据库中：

    // Retrieve the flight by the attributes, or create it if it doesn't exist...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // Retrieve the flight by the attributes, or instantiate a new instance...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

<a name="deleting-models"></a>
## 删除模型

你可以在模型实例中使用 `delete` 方法来删除模型：

    $flight = App\Flight::find(1);

    $flight->delete();

#### 根据主键删除已存在的模型

在上面的例子中，我们先从数据库中检索出相关的模型，然后才调用 `delete` 方法来进行删除。事实上，如果你知道了模型的主键，那么你完全可以不用去检索到它。你可以直接使用 `destroy` 方法来进行删除：

    App\Flight::destroy(1);

    App\Flight::destroy([1, 2, 3]);

    App\Flight::destroy(1, 2, 3);

#### 通过查询删除模型

当然，你也可以通过查询删除一个模型集。在这个例子中，我们将删除所有标记为未启用的航班。类似于集体更新，集体删除并不会触发被删除模型的模型事件。

    $deletedRows = App\Flight::where('active', 0)->delete();

> {note} 当通过 Eloquent 执行集体删除操作时，被删除的模型的 `deleting` 和 `deleted` 模型事件并不会被触发。这是因为这些模型在删除时并没有被检索过。

<a name="soft-deleting"></a>
### 软删除

除了从数据库中真实的删除数据之外，Eloquent 也可以进行软删除操作。当模型是一个软删除模型时，它们并不会真正的从数据库中清除记录。实际上，它们会将模型的 `deleted_at` 属性进行设置，并且将其更新到数据库中。如果模型中的 `deleted_at` 的值不是 `NULL`，那么它就被标记为软删除了。如果你需要启用软删除模型，你需要在模型中引入 `Illuminate\Database\Eloquent\SoftDeletes` trait，并且在模型的 `$dates` 属性中添加 `deleted_at` 列：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;

        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

当然，你应该在数据表中添加 `deleted_at` 列。Laravel 的 [结构生成器](/docs/{{version}}/migrations) 中提供了生成该列的方法:

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });

现在，当我们调用 `delete` 方法时，`deleted_at` 列会被设置为当前时间。并且，当查询启用软删除的模型时，已经被软删除的模型将自动从所有的查询结果中剔除。

你可以使用 `trashed` 方法来判断所给定的模型是否已经被软删除：

    if ($flight->trashed()) {
        //
    }

<a name="querying-soft-deleted-models"></a>
### 查询软删除的模型

#### 包含软删除的模型

就如上面我们所提到的，被软删除的模型将自动的从结果中进行剔除。事实上，你可以使用 `withTrashed` 方法来强制结果中显示已经被软删除的模型：

    $flights = App\Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

`whitTrashed` 方法也可以在 [关联](/docs/{{version}}/eloquent-relationships) 查询中进行使用：

    $flight->history()->withTrashed()->get();

#### 只检索软删除的模型

`onlyTrashed` 方法会从数据库中检索被软删除的模型记录：

    $flights = App\Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

#### 还原软删除的模型

有时候你可能会希望还原已经被软删除的模型，你可以使用 `restore` 方法来将模型从软删除中解除：

    $flight->restore();

你也可以在查询中使用 `restore` 方法来快速的重启多个模型。类似其它集体操作一样，这不会触发模型中的任何事件：

    App\Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

就像 `withTrashed` 方法一样，`restore` 方法也可以在 [关联](/docs/{{version}}/eloquent-relationships) 查询中使用：

    $flight->history()->restore();

#### 永久删除模型

有时候你可能希望从数据库中直接删除这个模型。你可以使用 `forceDelete` 方法来将模型从数据库中永久的删除：

    // Force deleting a single model instance...
    $flight->forceDelete();

    // Force deleting all related models...
    $flight->history()->forceDelete();

<a name="query-scopes"></a>
## 区间查询

<a name="global-scopes"></a>
### 全局区间

全局区间允许你在给定的模型上的所有查询进行约束的添加。Laravel 自己的 [软删除](#soft-deleting) 功能就是利用了全局区间来从数据库中拉取未删除的数据。编写自己的全局区间可以方便的对给定模型的所有查询进行约束。

#### 编写全局区间

编写一个全局区间非常简单，你只需要定义一个实现了 `Illuminate\Database\Eloquent\Scope` 接口的类。这个接口只要求你实现一个方法：`apply`。`apply` 方法可以在需要的时添加 `where` 约束：

    <?php

    namespace App\Scopes;

    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class AgeScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            return $builder->where('age', '>', 200);
        }
    }

> {tip} Laravel 没有为区间预置存放的目录，所以你可以自由的创建自己的 `Scopes` 目录来进行管理。

#### 应用全局区间

你需要在给定的模型中复写 `boot` 方法并且使用 `addGlobalScope` 方法来分配全局区间：

    <?php

    namespace App;

    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope(new AgeScope);
        }
    }

在添加完区间之后，调用 `User::all()` 的查询会产生下述的 SQL：

    select * from `users` where `age` > 200

#### 匿名全局区间

Eloquent 也允许你通过一个闭包来定义全局区间，这通常对于不需要分离到单独一个类文件中的简单区间尤其有用：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;

    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();

            static::addGlobalScope('age', function(Builder $builder) {
                $builder->where('age', '>', 200);
            });
        }
    }

传递到 `addGlobalScope()` 方法的首个参数将作为区间的唯一标识，你可以通过标识将其排除：

    User::withoutGlobalScope('age')->get();

#### 删除全局区间

如果你希望从给定的查询中移除全局区间，你可以使用 `withoutGlobalScope` 方法。这个方法接收全局区间的类名作为唯一参数：

    User::withoutGlobalScope(AgeScope::class)->get();

如果你希望删除多个或者全部的全局区间，你可以使用这么使用 `withoutGlobalScopes` 方法：

    // Remove all of the global scopes...
    User::withoutGlobalScopes()->get();

    // Remove some of the global scopes...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### 当前区间

当前区间允许你在模型中定义一些可以复用的常用的约束。比如，你可能经常需要检索一些受欢迎的用户。你可以简单的在模型方法中前置 `scope` 命名来定义一个区间.

区间应该总是返回一个查询生成器的实例：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }

        /**
         * Scope a query to only include active users.
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

#### 使用当前区间

一旦区间进行了定义，你可以在模型进行查询时调用区间方法，事实上，你在调用区间方法时并不需要包含 `scope` 前缀。你甚至可以链式的调用其它的区间：

    $users = App\User::popular()->active()->orderBy('created_at')->get();

#### 动态区间

有时候你可能希望定义一个可以接收参数的区间。在开始之前，你仅仅只需要在你的区间中添加一些额外的参数。区间参数应该在 `$query` 参数之后进行定义：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Scope a query to only include users of a given type.
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeOfType($query, $type)
        {
            return $query->where('type', $type);
        }
    }

现在，你可以在调用区间时传递一些参数了：

    $users = App\User::ofType('admin')->get();

<a name="events"></a>
## 事件

Eloquent 模型可以触发多种事件，这允许你在模型的生命周期的各个关键点进行 hook 操作。你可以使用下面的方法进行 Hook：`creating`，`created`，`updating`，`updated`，`saving`，`saved`，`deleting`，`deleted`，`restoring`，`restored`。事件允许你轻松的在模型进行存储或更新操作时进行执行额外的操作。

当一个新的模型首次进行存储操作时，会触发 `creating` 和 `created` 事件。如果模型已经存在于数据库中，并且调用 `save` 方法，那么 `updating` / `updated` 事件将会被触发。事实上，在这两种情况下，`saving` 和 `saved` 事件都会被触发。

举个示例，让我们在 [服务提供者](/docs/{{version}}/providers) 中定义一个 Eloquent 事件监听器。在我们的事件监听器中，我们将在给定的模型中调用 `isValid` 方法，当模型并没有通过验证时将返回 `false`。如果从 Eloquent 事件监听器中返回 `false`，那么将取消 `save` / `update` 操作：

    <?php

    namespace App\Providers;

    use App\User;
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
            User::creating(function ($user) {
                return $user->isValid();
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

<a name="observers"></a>
### 观察者

如果你需要在给定的模型上监听多种事件，那么你可以使用观察者来将这些监听器集中到一个单独的类中。观察者类中的方法名反映你需要监听的 Eloquent 事件。所有的这些方法都接收模型实例作为它们的唯一参数。Laravel 并没有为观察者提供默认的存储目录，你可以自由的创建一个目录来存储观察者类：

    <?php

    namespace App\Observers;

    use App\User;

    class UserObserver
    {
        /**
         * Listen to the User created event.
         *
         * @param  User  $user
         * @return void
         */
        public function created(User $user)
        {
            //
        }

        /**
         * Listen to the User deleting event.
         *
         * @param  User  $user
         * @return void
         */
        public function deleting(User $user)
        {
            //
        }
    }

你还需要在你想要观察的模型中调用 `observe` 方法来进行观察者的注册。你可以在某一个服务提供者的 `boot` 方法中注册观察者。在这个例子中，我们将会在 `AppServiceProvider` 中注册观察者：

    <?php

    namespace App\Providers;

    use App\User;
    use App\Observers\UserObserver;
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
            User::observe(UserObserver::class);
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
