# 数据库: 查询生成器

- [前言](#introduction)
- [检索结果](#retrieving-results)
    - [结果分块](#chunking-results)
    - [聚合](#aggregates)
- [查询](#selects)
- [原生表达式](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [Where 子句](#where-clauses)
    - [组合参数](#parameter-grouping)
    - [Where Exists 子句](#where-exists-clauses)
    - [JSON Where 子句](#json-where-clauses)
- [Ordering, Grouping, Limit, & Offset](#ordering-grouping-limit-and-offset)
- [条件子句](#conditional-clauses)
- [Inserts](#inserts)
- [Updates](#updates)
    - [更新 JSON 列](#updating-json-columns)
    - [增量 & 减量](#increment-and-decrement)
- [Deletes](#deletes)
- [悲观锁](#pessimistic-locking)

<a name="introduction"></a>
## 前言

Laravel 的数据库查询生成器提供了一种方便流利的接口来构建和运行数据库查询。它可以用来在应用中提供执行大多数的数据库操作，并且它可以在所有支持的数据库系统中进行协作。

Laravel 的查询生成器使用了 PDO 参数绑定来针对可能的 SQL 注入攻击。所以这里不需要手动的去清洗作为绑定所传递的字符串。

<a name="retrieving-results"></a>
## 检索结果

#### 从表中检索所有行

在开始流畅的执行查询之前，你需要先使用 `DB` 假面的 `table` 方法，`table` 方法会返回一个给定表的查询生成器实例，这允许你链式添加多种查询约束，并最终使用 `get` 方法获取结果:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

就像原生的查询一样，`get` 方法会返回一个 `Illuminate\Support\Collection` 实例包含结果集，其每一项结果都是一个 `StdClass` PHP 类的对象。你可以通过其属性来访问每一列的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 从表中检索一个单独的行或者列

如果你只需要从数据表中检索出单独的一行，你可以使用 `first` 方法，该方法会返回一个单独的 `StdClass` 对象：

    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;

如果你甚至不需要一个完整的行，那么你可以通过使用 `value` 方法从记录中获取一个精确的列的值。该方法会直接返回列中的值：

    $email = DB::table('users')->where('name', 'John')->value('email');

#### 检索多列的值

如果你希望获取一个数组来包含一个列中所有的值，你可以使用 `pluck` 方法，在这个例子中，我们将获取所有的角色名称：

    $titles = DB::table('roles')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

你也可以为所返回的数组指定自定义的键：

    $roles = DB::table('roles')->pluck('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### 对结果进行分块

如果你与表中的数千条记录协作，你可以考虑使用 `chunk` 方法，该方法一次会从结果中检索出一小块。并且将这每一块放进闭包中提供给进程。该方法对于编写 [Artisan commands](/docs/{{version}}/artisan) 命令来处理数千条的记录时非常有用。比如，让我们将整个用户表分块成每次 100 条来工作：

    DB::table('users')->orderBy('id')->chunk(100, function($users) {
        foreach ($users as $user) {
            //
        }
    });

你可以通过在 `Closure` 中返回 `false` 来停止进一步的处理块的操作：

    DB::table('users')->orderBy('id')->chunk(100, function($users) {
        // Process the records...

        return false;
    });

<a name="aggregates"></a>
### 聚合方法

查询生成器也提供了多种聚合方法。比如 `count`，`max`，`min`，`avg`，和 `sum`。你可以在查询构建后调用这些方法：

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

当然，你可以结合其它约束来构建你的查询：

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="selects"></a>
## Selects

#### 指定一个选择约束

当然，你不会总是想要从数据库中获取所有的列。你可以使用 `select` 方法来指定一个自定义的 `select` 约束到查询：

    $users = DB::table('users')->select('name', 'email as user_email')->get();

`distinct` 方法允许你强迫查询返回一个不重复的结果：

    $users = DB::table('users')->distinct()->get();

如果你已经获取了查询生成器的实例，并且你想要在已经添加过选择约束的实例中添加额外一列的检索，你可以使用 `addSelect` 方法：

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## 原生表达式

有时候，你可能需要在查询时使用原生的表达式，这些表达式会以字符串的方式注入到查询中，所以你应该小心的不要构造出任何的 SQL 注入点。你可以使用 `DB::raw` 方法来构造一个原生的表达式：

    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();

<a name="joins"></a>
## Joins

#### 内连接

查询生成器也可以用来编写 join 声明。为了执行一个基础的内连接，你可以在查询生成器中使用 `join` 方法。`join` 方法所接收的第一个参数应该是你需要加入的表的名称，而其它的参数则是对加入提供约束。当然，就如你所看到的，你可以在一个查询中加入多个表：

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

#### 左连接

如果你需要执行 `left join` 来代替 `inner join`，你可以使用 `leftJoin` 方法，`leftJoin` 方法具有和 `join` 方法相同的使用方法：

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

#### 交叉连接

你可以使用 `crossJoin` 方法来执行一个交叉的连接操作，你需要为 `crossJoin` 提供一个你想要交叉连接的表名。交叉连接会生成第一个表和加入的表的笛卡尔积：

    $users = DB::table('sizes')
                ->crossJoin('colours')
                ->get();

#### 高级连接子句

你也可以指定更高级的 join 约束。你可以传递一个 `Closure` 作为 `join` 方法的第二个参数。`Closure` 会接收一个 `JoinClause` 对象并且该对象允许你指定 `join` 约束：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();

如果你喜欢在 joins 中使用 `where` 风格约束，那么你可以在 join 上使用 `where` 和 `orWhere` 方法。该方法会比较列的值而取代直接比较两列：

    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="unions"></a>
## Unions（联合）

查询生成器也提供了一种快捷的方式来将两个查询联合起来。比如，你可以构建一个初始化的查询，然后使用 `union` 方法来将其与之后的查询联合起来：

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

> {tip} `unionAll` 方法也是可用的，并且它与 `union` 方法具有相同的使用方式。

<a name="where-clauses"></a>
## Where 子句

#### 简单的 Where 子句

你可以在查询生成器中使用 `where` 方法来在查询中添加 `where` 约束。大多数基础的 `where` 调用都需要提供三个参数，第一个参数是列的名称，第二个参数是比较操作的方式，第三个参数则是针对列所需要评估的值。

比如，这里的查询用来验证 "votes" 列的值等于 100:

    $users = DB::table('users')->where('votes', '=', 100)->get();

为了方便，你可以直接传递需要比较的值到第二个参数来表明列值与给定值相等的判断：

    $users = DB::table('users')->where('votes', 100)->get();

当然，你也可以在使用 `where` 子句时使用其它的操作符：

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

你也可以传递一个数组所组成的条件到 `where` 方法中：

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### Or 声明

你可以将 where 约束连在一起，以及添加 `or` 子句到查询中。`orWhere` 方法接收和 `where` 方法相同的参数：

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

#### 其它 Where 子句

**whereBetween**

`whereBetween` 方法确定列的值是否在两个给定值之间：

    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();

**whereNotBetween**

`whereNotBetween` 方法确定列的值应该不在所给定值区间内：

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn**

`whereIn` 方法验证给定的列的值应该是给定的数组值中之一：

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn` 方法验证所给定的列的值应该不在给定的数组中：

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull**

`whereNull` 方法验证所给定列的值应该是 `NULL`:

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull` 方法验证所给定的列的值不应该为 `NULL`:

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear**

`whereDate` 方法可以用来比较列中的值是否属于给定的日期：

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-10-10')
                    ->get();

`whereMonth` 方法可以用来比较列中的值是否属于指定的月份：

    $users = DB::table('users')
                    ->whereMonth('created_at', '10')
                    ->get();

`whereDay` 方法可以用来比较列中的值是否属于具体月份中的某一天:

    $users = DB::table('users')
                    ->whereDay('created_at', '10')
                    ->get();

`whereYear` 方法可以用来比较列中的值是否属于指定的年份：

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

**whereColumn**

`whereColumn` 方法可以用来判断两列的值是否相等：

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

你也可以传递一个比较符到方法中：

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

`whereColumn` 方法可以接收一个包含了多个限制条件的数组。这些条件会被使用 `add` 判定操作：

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at']
                    ])->get();

<a name="parameter-grouping"></a>
### 参数分组

有时候你需要构建一些更加高级的 where 子句，比如，"where exists" 或者嵌套的参数分组。Laravel 的查询生成器也能很好的处理这些。让我们来看一个分组括号内约束的例子：

    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();

就如你所看到的，传递一个 `Closure` 到 `orWhere` 方法来指导查询生成器开始一个组约束。`Closure` 会接收一个查询生成器的实例，这样你就可以在括号组内构建一些约束。上面的示例将会生成下面的 SQL：

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

<a name="where-exists-clauses"></a>
### Where Exists 子句

`whereExists` 方法允许你编写 `where exists` SQL 子句。`whereExists` 方法接收一个 `Closure` 参数，这个闭包会接收一个查询生成器实例，这允许你在 "exists" 子句中去构建查询：

    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();

上面的查询将生成下面的 SQL:

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

<a name="json-where-clauses"></a>
### JSON Where 子句

Laravel 也可以对支持 JSON 列类型的数据库进行 JSON 类型的列的查询。目前，这只包含在 MySQL 5.7 和 Postgres。你需要使用 `->` 操作符来查询 JSON 列：

    $users = DB::table('users')
                    ->where('options->language', 'en')
                    ->get();

    $users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## 排序，分组，限制 & 位移

#### orderBy

`orderBy` 方法允许你对给定列的查询结果进行排序。`orderBy` 所接受的第一个参数应该是你希望参照排序的列，而第二个参数应该是你决定进行排序的方向，它们应该是 `asc` 或者 `desc`:

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### inRandomOrder

`inRandomOrder` 方法可以用来对查询的结果进行随机排序。比如，你可以使用这个方法来随机的获取用户：

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having / havingRaw

`groupBy` 和 `having` 方法可以用来对查询结果进行分组。`having` 方法具有和 `where` 方法相同的使用方式：

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

`havingRaw` 方法可以用来在 `having` 子句中使用原生的字符串值，比如，我们可以找到所有销售额大于 $2,500 的部门：

    $users = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();

#### skip / take

你可以使用 `take` 和 `skip` 方法来限制查询所返回结果的数量或者在查询中跳过一定的数目（`offset`）：

    $users = DB::table('users')->skip(10)->take(5)->get();

<a name="conditional-clauses"></a>
## 条件子句

有时候你可能希望只有当某些条件达成时才去添加一些查询到语句中。又或者你想要在只有进入的请求数据中含有指定的字段时才会判断在 `where` 子句中进行加入。你可以使用 `when` 方法来做到这些：

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function ($query) use ($role) {
                        return $query->where('role_id', $role);
                    })
                    ->get();


`when` 方法只有所给定的第一个参数为 `true` 时才会执行给定的闭包。如果第一个参数是 `false`，那么闭包将不会被执行。

<a name="inserts"></a>
## Inserts

查询生成器也为插入记录到数据库中提供了 `insert` 方法。`insert` 方法可以接受列名所组成的键值对数组作为参数：

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );

你也可以在一次调用 `insert` 方法时添加多条记录到数据库中，你需要传递一个嵌套的数组，数组中的每一个数组都代表了数据库中的一行：

    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

#### 自动递增的 IDs

如果挑中存在自动递增的 id，你可以使用 `insertGetId` 方法来插入记录的同时获取 ID：

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> {note} 当使用 PostgreSQL 时，insertGetId 方法期望自增的列名为 `id`。如果你想要检索的 ID 是一个其他列名，你需要传递列名到 `insertGetId` 方法的第二个参数。

<a name="updates"></a>
## Updates

当然，除了新增记录之外，查询生成器也提供了为数据库中存在的记录更新的 `update` 方法。`update` 方法像 `insert` 方法一样，接收一个键值对所组成的数组来进行列值的更新。你可以在使用 `update` 查询时使用 `where` 子句：

    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);

<a name="updating-json-columns"></a>
### 更新 JSON 类型的列

当更新 JSON 类型的列时，你应该使用 `->` 语法来访问 JSON 对象中适当的键。这个操作语法只对哪些支持 JSON 类型列的数据库有效：

    DB::table('users')
                ->where('id', 1)
                ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### 增量 & 递减

查询生成器也提供一些方便的方法来对列的值进行增量和递减。这只是一种快捷的方式，用来提供比手动写入 `update` 语句更具表现力和简洁的接口。

这两种方法都接收最少一个参数：待修改的列名，第二个参数是一个可选的参数，它用来控制递增或者递减的数量：

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

你也可以指定同时更新额外的列：

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## Deletes

当然，查询生成器也提供了从数据库中删除记录的 `delete` 方法。你可以在调用 `delete` 方法之前添加一些 `where` 子句：

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

如果你希望清除整个表的数据，你可以使用 `truncate` 方法，它将清除所有的行并且重置自增的 ID 到 0：

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲观锁

查询生成器也提供了一些方法来帮助你在 `select` 声明中实现 "悲观锁"。你可以在查询中使用 `sharedLock` 方法来运行 "共享锁"。共享锁可以保证所选定的行在事务提交前不会被修改：

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

另外，你也可以使用 `lockForUpdate` 方法。更新锁可以防止行被修改或者被另一共享锁所选中：

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
