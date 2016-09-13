# 帮助方法

- [前言](#introduction)
- [可用的方法](#available-methods)

<a name="introduction"></a>
## 前言

Laravel 包含了多中 "帮手" PHP 函数，很多方法都在框架中进行了使用，如果你发现他们很方便，你也可以在自己的应用中使用。

<a name="available-methods"></a>
## 可用的方法

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### 数组

<div class="collection-method-list" markdown="1">

[array_add](#method-array-add)
[array_collapse](#method-array-collapse)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_has](#method-array-has)
[array_last](#method-array-last)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_prepend](#method-array-prepend)
[array_pull](#method-array-pull)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_sort_recursive](#method-array-sort-recursive)
[array_where](#method-array-where)
[head](#method-head)
[last](#method-last)
</div>

### 路径

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[elixir](#method-elixir)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### 字符串

<div class="collection-method-list" markdown="1">

[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[snake_case](#method-snake-case)
[str_limit](#method-str-limit)
[starts_with](#method-starts-with)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[studly_case](#method-studly-case)
[title_case](#method-title-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### URLs

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[secure_asset](#method-secure-asset)
[route](#method-route)
[url](#method-url)

</div>

### 其它

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[collect](#method-collect)
[config](#method-config)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[dispatch](#method-dispatch)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[method_field](#method-method-field)
[old](#method-old)
[redirect](#method-redirect)
[request](#method-request)
[response](#method-response)
[session](#method-session)
[value](#method-value)
[view](#method-view)

</div>

<a name="method-listing"></a>
## 方法列表

<style>
    #collection-method code {
        font-size: 14px;
    }

    #collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="arrays"></a>
## 数组

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

`array_add` 方法用来在数组中添加键值对，它仅会在数组中不存在所给定的键时才会添加：

    $array = array_add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `array_collapse()` {#collection-method}

`array_collapse` 方法将会瓦解一个数组中的多个数组到一个单一的数组中。

    $array = array_collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

`array_divide` 方法将会分列数组，它会返回两个数组，一个数组包含了原数组的所有的键，另一个数组包含原数组所有的值：

    list($keys, $values) = array_divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}

`array_dot` 方法将数组从多维降低为一维数组，它使用 `.` 符号来表明其深度：

    $array = array_dot(['foo' => ['bar' => 'baz']]);

    // ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except` 方法从数组中移除指定的键值对：

    $array = ['name' => 'Desk', 'price' => 100];

    $array = array_except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first` 方法返回数组回调迭代中第一个返回真值的元素:

    $array = [100, 200, 300];

    $value = array_first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

你也可以在第三个参数中传递一个默认值，如果迭代结束仍未返回真值，将返回默认值:

    $value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten` 方法会将多维数组降为一维数组：

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $array = array_flatten($array);

    // ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget` 方法可以使用 `.` 语法来删除数组中嵌套的键值对:

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get` 方法可以使用 `.` 语法来从数组中检索嵌套的值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $value = array_get($array, 'products.desk');

    // ['price' => 100]

`array_get` 方法也可以接收第三个参数，用来作为默认值，如果数组中并没有检索到相应的值，将会返回默认值：

    $value = array_get($array, 'names.john', 'default');

<a name="method-array-has"></a>
#### `array_has()` {#collection-method}

`array_has` 方法允许使用 `.` 语法来检查数组中是否含有给定的项：

    $array = ['products' => ['desk' => ['price' => 100]]];

    $hasDesk = array_has($array, 'products.desk');

    // true

<a name="method-array-last"></a>
#### `array_last()` {#collection-method}

`array_last` 方法返回数组回调迭代中最后一个通过真值测试的元素:

    $array = [100, 200, 300, 110];

    $value = array_last($array, function ($value, $key) {
        return $value >= 150;
    });

    // 300

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

`array_only` 方法会从给定的数组中返回指定的键值对：

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $array = array_only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck` 方法摘取数组中所给定的键值对中的值：

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $array = array_pluck($array, 'developer.name');

    // ['Taylor', 'Abigail'];

你也可以指定返回的结果如何键化：

    $array = array_pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail'];

<a name="method-array-prepend"></a>
#### `array_prepend()` {#collection-method}

`array_prepend` 方法会在数组的起始端加入一项：

    $array = ['one', 'two', 'three', 'four'];

    $array = array_prepend($array, 'zero');

    // $array: ['zero', 'one', 'two', 'three', 'four']

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

`array_pull` 方法从数组中返回键值对并将其在数组中进行剔除：

    $array = ['name' => 'Desk', 'price' => 100];

    $name = array_pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set` 方法使用 `.` 语法对数组中的项进行设置值：

    $array = ['products' => ['desk' => ['price' => 100]]];

    array_set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort` 方法会根据给定闭包所返回的结果对数组进行排序：

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Chair'],
    ];

    $array = array_values(array_sort($array, function ($value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `array_sort_recursive()` {#collection-method}

`array_sort_recursive` 方法会对数组进行递归的使用 `sort` 方法排序：

    $array = [
        [
            'Roman',
            'Taylor',
            'Li',
        ],
        [
            'PHP',
            'Ruby',
            'JavaScript',
        ],
    ];

    $array = array_sort_recursive($array);

    /*
        [
            [
                'Li',
                'Roman',
                'Taylor',
            ],
            [
                'JavaScript',
                'PHP',
                'Ruby',
            ]
        ];
    */

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

`array_where` 方法会根据给定的闭包对数组进行过滤：

    $array = [100, '200', 300, '400', 500];

    $array = array_where($array, function ($value, $key) {
        return is_string($value);
    });

    // [1 => 200, 3 => 400]

<a name="method-head"></a>
#### `head()` {#collection-method}

`head` 方法简单的从数组中返回其首个元素：

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 方法返回所给定数组中的最后一个元素：

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## 路径

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path` 方法返回 `app` 目录的完整路径。你也可以使用 `app_path` 方法来生成相对于应用目录的完整路径：

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path` 方法返回项目根目录的完整路径。你也可以使用 `base_path` 方法来返回相对于根目录的完整路径：

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path` 方法用来返回应用的配置文件目录的完整路径：

    $path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path` 方法返回应用的数据库目录的完整路径：

    $path = database_path();

<a name="method-elixir"></a>
#### `elixir()` {#collection-method}

`elixir` 方法返回 [版本化](/docs/{{version}}/elixir) 的文件路径：

    elixir($file);

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path` 方法返回 `public` 目录的完整路径：

    $path = public_path();

<a name="method-resource-path"></a>
#### `resource_path()` {#collection-method}

`resource_path` 方法返回完整的 `resources` 目录路径。你也可以使用 `resource_path` 方法来生成相对于 `resources` 目录的完整路径：

    $path = resource_path();

    $path = resource_path('assets/sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path` 方法返回 `storage` 目录的完整路径，你也可以使用 `storage_path` 来生成相对于 `storage` 目录的完整路径：

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="strings"></a>
## 字符串

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case` 方法将给定字符串转换成 `camelCase` 格式：

    $camel = camel_case('foo_bar');

    // fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` 反回所给定类移除命名空间之后的类名：

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

`e` 方法使用 `htmlentities` 方法来过滤给定字符串：

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

`ends_with` 方法用来判断给定的字符串是否以给定的值结尾：

    $value = ends_with('This is my name', 'name');

    // true

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

`snake_case` 方法将字符串转换成 `snake_case` 格式：

    $snake = snake_case('fooBar');

    // foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

`str_limit` 方法用来限制字符串的长度。该方法的第一个参数应该是一个字符串，而第二个参数应该是允许返回结果的最大长度：

    $value = str_limit('The PHP framework for web artisans.', 7);

    // The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with` 方法用来判断给定的字符串是否已给定的值起始：

    $value = starts_with('This is my name', 'This');

    // true

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains` 方法判断给定的字符串中是否包含给定的值：

    $value = str_contains('This is my name', 'my');

    // true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish` 方法可以在字符串的结尾添加独特的值（如果字符串不是以该值结尾）：

    $string = str_finish('this/string', '/');

    // this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

`str_is` 方法用来判断给定的字符串是否匹配给定的模式。可以使用星号作为通配符：

    $value = str_is('foo*', 'foobar');

    // true

    $value = str_is('baz*', 'foobar');

    // false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

`str_plural` 方法将字符串转换为其相应的复数形式，该方法目前只支持英语：

    $plural = str_plural('car');

    // cars

    $plural = str_plural('child');

    // children

你可以传递一个整型值到第二个参数来表明返回单数或复数形式：

    $plural = str_plural('child', 2);

    // children

    $plural = str_plural('child', 1);

    // child

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random` 方法根据指定的长度生成随机字符串，这个方法使用 PHP 的 `random_bytes` 方法：

    $string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular` 方法将字符串转换为单数形式，目前只支持英语：

    $singular = str_singular('cars');

    // car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug` 方法使用给定的胶连字符将给定的字符串生成 URL:

    $title = str_slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case` 方法转换给定的字符串到 `StudlyCase` 格式：

    $value = studly_case('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `title_case()` {#collection-method}

`title_case` 方法转换给定的字符串到 `Title Case` 格式：

    $title = title_case('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans` 方法根据 [本地化文件](/docs/{{version}}/localization) 中的语言行来进行翻译：

    echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice` 方法来转译到给定的语言，并使用相应的单复数形式：

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

`action` 方法根据给定的控制器动作生成相应的 URL。你不需要传递完整的命名空间。默认的所传递的控制器类名是相对于 `App\Http\Controllers` 的命名空间：

    $url = action('HomeController@getIndex');

如果方法接受路由参数，你可以传递第二个参数到该方法：

    $url = action('UserController@profile', ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {#collection-method}

`asset` 方法根据当前的请求协议（HTTP or HTTPS）来返回指定资源的地址：

	$url = asset('img/photo.jpg');

<a name="method-secure-asset"></a>
#### `secure_asset()` {#collection-method}

使用 HTTPS 生成给定资源的 URL：

	echo secure_asset('foo/bar.zip', $title, $attributes = []);

<a name="method-route"></a>
#### `route()` {#collection-method}

`route` 方法根据给定的路由名称来生成 URL：

    $url = route('routeName');

如果路由接受参数，你可以传递第二个参数到方法：

    $url = route('routeName', ['id' => 1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url` 方法根据指定的路径生成完整的路径：

    echo url('user/profile');

    echo url('user/profile', [1]);

如果没有路径指定，将返回 `Illuminate\Routing\UrlGenerator` 的实例：

    echo url()->current();
    echo url()->full();
    echo url()->previous();

<a name="miscellaneous"></a>
## 其它

<a name="method-abort"></a>
#### `abort()` {#collection-method}

`abort` 方法会抛出一个 HTTP 异常，它会经过异常处理器进行渲染:

    abort(401);

你也可以提供异常的响应文本：

    abort(401, 'Unauthorized.');

<a name="method-abort-if"></a>
#### `abort_if()` {#collection-method}

`abort_if` 方法会在所给定的布尔表达式返回 `true` 时抛出一个 HTTP 异常：

    abort_if(! Auth::user()->isAdmin(), 403);

<a name="method-abort-unless"></a>
#### `abort_unless()` {#collection-method}

`abort_unless` 方法会在所给定的布尔表达式返回 `false` 时抛出一个异常：

    abort_unless(Auth::user()->isAdmin(), 403);

<a name="method-auth"></a>
#### `auth()` {#collection-method}

`auth` 方法返回一个认证器实例。你可以方便的使用它来替换 `Auth` 假面：

    $user = auth()->user();

<a name="method-back"></a>
#### `back()` {#collection-method}

`back` 方法生成重定向响应到用户之前的地址：

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {#collection-method}

`bcrypt` 方法使用 Bcrypt 加密来哈希化给定的值。你也可以通过 `Hash` 假面来调用：

    $password = bcrypt('my-secret-password');

<a name="method-collect"></a>
#### `collect()` {#collection-method}

`collect` 方法根据给定的数组来生成 [集合](/docs/{{version}}/collections) 实例：

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {#collection-method}

`config` 根据给定的值来获取配置项的值。你可以使用 `.` 语法来获取配置项的值。也可以传递第二个参数作为配置项未找到时的默认值：

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

你也可以在运行时使用键值对的方式对配置进行设置：

    config(['app.debug' => true]);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

`csrf_field` 方法用来生成一个 `hidden` 文本框字段来包含 CSRF token。你可以在 [Blade](/docs/{{version}}/blade) 模板中使用：

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_token` 方法返回当前的 CSRF token 值：

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 方法打印输出给定的变量并终止执行之后的代码：

    dd($value);

    dd($value1, $value2, $value3, ...);

如果你不想停止执行之后的代码，你应该使用 `dump` 方法：

    dump($value);

<a name="method-dispatch"></a>
#### `dispatch()` {#collection-method}

`dispatch` 方法在 Laravel 的 [任务队列](/docs/{{version}}/queues) 中添加一个新的任务：

    dispatch(new App\Jobs\SendEmails);

<a name="method-env"></a>
#### `env()` {#collection-method}

`env` 方法用来获取环境变量，也可以在未设置环境变量时返回默认值：

    $env = env('APP_ENV');

    // Return a default value if the variable doesn't exist...
    $env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event` 方法分发给定的 [事件](/docs/{{version}}/events) 到它的监听器中：

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {#collection-method}

`factory` 方法创建一个模型工厂构造器来生成给定的类名的实例。你可以在写 [测试](/docs/{{version}}/database-testing#writing-factories) 或者 [播种](/docs/{{version}}/seeding#using-model-factories) 时使用：

    $user = factory(App\User::class)->make();

<a name="method-method-field"></a>
#### `method_field()` {#collection-method}

`method_field` 方法生成一个 `hidden` 文本框来包含一个欺骗性的 HTTP 请求动词。你可以在 [Blade](/docs/{{version}}/blade) 模板中使用：

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-old"></a>
#### `old()` {#collection-method}

`old` 方法 [检索](/docs/{{version}}/requests#retrieving-input) session 中闪存的旧的文本值：

    $value = old('value');

    $value = old('value', 'default');

<a name="method-redirect"></a>
#### `redirect()` {#collection-method}

`redirect` 方法返回一个重定向 HTTP 响应，或者在无参数调用时返回一个重定向器实例：

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-request"></a>
#### `request()` {#collection-method}

`request` 方法返回当前的 [请求](/docs/{{version}}/requests) 实例或者检索请求中的输入项：

    $request = request();

    $value = request('key', $default = null)

<a name="method-response"></a>
#### `response()` {#collection-method}

`response` 方法生成一个 [响应](/docs/{{version}}/responses) 实例或者从响应工厂中获得一个实例：

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-session"></a>
#### `session()` {#collection-method}

`session` 方法可以用来获取或者设置 session 值：

    $value = session('key');

你可以通过传递键值对来进行 session 值的设置：

    session(['chairs' => 7, 'instruments' => 3]);

如果调用的是无参数的 `session` 方法，将返回 session 存储器实例：

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-value"></a>
#### `value()` {#collection-method}

`value` 方法会简单的返回所给定的值。但是，如果你传递的是一个 `Closure` ，`Closure` 将会被执行，其结果将被返回：

    $value = value(function() { return 'bar'; });

<a name="method-view"></a>
#### `view()` {#collection-method}

`view` 方法用来检索 [视图](/docs/{{version}}/views) 实例：

    return view('auth.login');
