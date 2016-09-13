# 集合

- [前言](#introduction)
    - [创建集合](#creating-collections)
- [可用的方法](#available-methods)

<a name="introduction"></a>
## 前言

`Illuminate\Support\Collection` 类为数组数据的流利操作提供了许多有用的方法。比如，看下面的演示代码。我们会使用 `collect` 帮助方法从一个数组中创建一个新的集合实例，然后运行 `strtoupper` 方法将所有元素转为大写之后再剔除集合中的空元素：

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });


就如你所看到的，`Collection` 类允许你链式的调用它的方法，就是这样提供了一种流利的映射的执行能力的同时缩小了底层的数组。通常情况下，所有的 `Collection` 方法都会返回一个全新的 `Collection` 实例。

<a name="creating-collections"></a>
### 创建集合

就如上面的代码，`collect` 帮助方法会根据给定的数组返回一个新的 `Illuminate\Support\Collection` 实例。所以，创建一个集合是非常的简单：

    $collection = collect([1, 2, 3]);

> {tip} [Eloquent](/docs/{{version}}/eloquent) 查询总是返回 `Collection` 的实例。

<a name="available-methods"></a>
## 可用的方法

在文档的余下部分，我们将探讨 `Collection` 类中所有可用的方法。你应该记住，所有的这些方法都可以链式调用来流利的操作底层的数组。另外，几乎每一个方法都会返回一个新的 `Collection` 实例，这允许你可以在必要时保存原始的集合。

<style>
    #collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[all](#method-all)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[combine](#method-combine)
[contains](#method-contains)
[count](#method-count)
[diff](#method-diff)
[diffKeys](#method-diffkeys)
[each](#method-each)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[isEmpty](#method-isempty)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[map](#method-map)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[merge](#method-merge)
[min](#method-min)
[only](#method-only)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[slice](#method-slice)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[splice](#method-splice)
[sum](#method-sum)
[take](#method-take)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[values](#method-values)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereIn](#method-wherein)
[whereInLoose](#method-whereinloose)
[zip](#method-zip)

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

<a name="method-all"></a>
#### `all()` {#collection-method .first-collection-method}

`all` 方法会简单的返回集合中所包含的底层数组:

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-avg"></a>
#### `avg()` {#collection-method}

`avg` 方法会返回集合中所有项的平均值：

    collect([1, 2, 3, 4, 5])->avg();

    // 3

如果集合中包含的是嵌套的数组或者对象，你应该传递 key 来表明所需要计算的平均值：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->avg('pages');

    // 636

<a name="method-chunk"></a>
#### `chunk()` {#collection-method}

`chunk` 方法会根据给定的大小来将集合分割成多个小的集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

该方法在 [视图](/docs/{{version}}/views) 中使用类似于 [Bootstrap](http://getbootstrap.com/css/#grid) 的网格系统时特别有用。想象一下，你有一个关于 [Eloquent](/docs/{{version}}/eloquent) 模型的集合想要在网格中显示：

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

<a name="method-collapse"></a>
#### `collapse()` {#collection-method}

`collapse` 方法会将集合中的底层数组中的多个数组瓦解到一个单一的数组集合中：

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-combine"></a>
#### `combine()` {#collection-method}

`combine` 方法会将一个集合中的值作为 keys 和另外一个集合或数组中的 values 相结合：

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-contains"></a>
#### `contains()` {#collection-method}

`contains` 方法来判断集合中是否含有给定的项：

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

你也可以传递键值对到 `contains` 方法，这将会用来判断集合中是否含有给定的键值对项:

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

最后，你也可以传递一个匿名函数到 `contains` 方法来提供自己的真值判断逻辑：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function ($value, $key) {
        return $value > 5;
    });

    // false

<a name="method-count"></a>
#### `count()` {#collection-method}

`count` 方法会返回集合中项目的总数：

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-diff"></a>
#### `diff()` {#collection-method}

`diff` 方法用来比较集合中存在而其他集合或者原生 PHP 的数组不存在的值。这个方法将会返回存在于原集合中而不存在与所给定集合中的值：

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

<a name="method-diffkeys"></a>
#### `diffKeys()` {#collection-method}

`diffKeys` 方法用基于键的方式比较集合中存在而其他集合或原生 PHP 数组不存在的项。这个方法将会返回在原集合中存在而在给定的集合中不存在的键值对:

    $collection = collect([
        'one' => 10,
        'two' => 20,
        'three' => 30,
        'four' => 40,
        'five' => 50,
    ]);

    $diff = $collection->diffKeys([
        'two' => 2,
        'four' => 4,
        'six' => 6,
        'eight' => 8,
    ]);

    $diff->all();

    // ['one' => 10, 'three' => 30, 'five' => 50]

<a name="method-each"></a>
#### `each()` {#collection-method}

`each` 方法会迭代集合中的每一项，并将该项传递给所给定的回调：

    $collection = $collection->each(function ($item, $key) {
        //
    });

在回调中返回 `false` 将会中断迭代：

    $collection = $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-every"></a>
#### `every()` {#collection-method}

`every` 方法会创建一个由每第 N 个元素（包含起始位）所组成的集合：

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->every(4);

    // ['a', 'e']

你可以传递第二个参数来设置位移：

    $collection->every(4, 1);

    // ['b', 'f']

<a name="method-except"></a>
#### `except()` {#collection-method}

`except` 方法返回集合中的项，并在项目中剔除选定的键：

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();

    // ['product_id' => 1]

如果想要获取与 `except` 相反的结果，请看 [only](#method-only) 方法。

<a name="method-filter"></a>
#### `filter()` {#collection-method}

`filter` 方法会根据给定的回调的迭代结果进行过滤，如果回调返回的是真值，该项将会被保留：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

如果想要获取与 `filter` 相反的结果，请看 [reject](#method-reject) 方法。

<a name="method-first"></a>
#### `first()` {#collection-method}

`first` 方法会返回集合中第一个在回调中返回真值的项:

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

你也可以调用无参数的 `first` 方法，该方法会返回集合中的第一个元素。如果集合是空的，则会返回 `null`：

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-flatmap"></a>
#### `flatMap()` {#collection-method}

`flatMap` 方法会迭代处理所有元素并返回新的元素进行替换，并将处理后的集合进行扁平化处理（如果项是数组，那么它将会瓦解平铺到集合中）：

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function ($values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {#collection-method}

`flatten` 方法将多维的集合转换为单一维度的集合：

    $collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

你也可以传递一个 "深度" 参数到方法：

    $collection = collect([
        'Apple' => [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ],
        'Samsung' => [
            ['name' => 'Galaxy S7', 'brand' => 'Samsung']
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

这里，调用不提供深度的 `flatten` 方法也会嵌套的扁平化数组，这将导致返回 `['iPhone 6S', 'Apple', 'GalaxyS7', 'Samsung']`。提供深度可以使你限制拉平嵌套数组的层级。

<a name="method-flip"></a>
#### `flip()` {#collection-method}

`flip` 方法将会反转键值对：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {#collection-method}

`forget` 方法根据所提供的键剔除集合中相应的项：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $collection->forget('name');

    $collection->all();

    // ['framework' => 'laravel']

> {note} 不像其他的集合方法，`forget` 方法不会返回一个新的修改后的集合，它会直接修改当前的集合。

<a name="method-forpage"></a>
#### `forPage()` {#collection-method}

`forPage` 方法将返回给定当前页所包含的项的集合。该方法需要传递当前页的数值及每页所包含的数目:

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {#collection-method}

`get` 方法将根据给定的键从集合中返回项，如果给定键不存在，则返回 `null`:

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

你可以传递第二个参数来作为默认值：

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('foo', 'default-value');

    // default-value

你也可以传递一个回调作为默认值。回调的结果将会被作为未找到键时的默认值：

    $collection->get('email', function () {
        return 'default-value';
    });

    // default-value

<a name="method-groupby"></a>
#### `groupBy()` {#collection-method}

`groupBy` 方法将会根据给定的键将集合进行分组：

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->toArray();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

除了传递一个字符串 `key` 之外，你也可以传递一个回调函数。回调函数应该返回你所期望进行分组的键的值：

    $grouped = $collection->groupBy(function ($item, $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->toArray();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

<a name="method-has"></a>
#### `has()` {#collection-method}

`has` 方法用来判断集合中是否存在给定的键:

    $collection = collect(['account_id' => 1, 'product' => 'Desk']);

    $collection->has('email');

    // false

<a name="method-implode"></a>
#### `implode()` {#collection-method}

`implode` 方法会将集合中的项连接成字符串。其参数取决于集合中的项目类型。如果集合中包含的是键值对数组或对象，你就应该传递一个键到方法来表明你想要连接的值，然后紧跟着一个胶连字符串参数：

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

如果集合只是包含了简单的字符串或者数组类型的值，你可以直接传递胶连字符参数到方法：

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {#collection-method}

`intersect` 方法返回给定数组与集合的交集，并且返回的结果将会使用原集合中的键：

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

<a name="method-isempty"></a>
#### `isEmpty()` {#collection-method}

`isEmpty` 方法用来判断集合是否为空，如果集合为空，则返回 `true`，否则返回 `false`：

    collect([])->isEmpty();

    // true

<a name="method-keyby"></a>
#### `keyBy()` {#collection-method}

`keyBy` 方法将会依据跟定的键对集合进行键化。如果多个项拥有相同的键，那么只有最后一项会出现在所返回的新的集合中：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'desk'],
        ['product_id' => 'prod-200', 'name' => 'chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

你也可以传递你自己的回调来返回集合键化所依赖的值：

    $keyed = $collection->keyBy(function ($item) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */


<a name="method-keys"></a>
#### `keys()` {#collection-method}

`keys` 方法返回集合中所有的键：

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 方法返回回调中最后一个返回真值的元素:

    collect([1, 2, 3, 4])->last(function ($value, $key) {
        return $value < 3;
    });

    // 2

你也可以调用无参数的 `last` 方法，它将返回集合中最后一个元素，如果集合为空，则返回 `null`:

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-map"></a>
#### `map()` {#collection-method}

`map` 方法会使用给定的回调来进行迭代集合中的所有项，在回调函数中可以自由的修改该项并进行返回，这样新的集合将包含修改后的值：

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function ($item, $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> {note} 就像其他的集合方法一样，`map` 返回一个新的集合实例。它并不会突变原集合。如果你想要在原有集合中进行改变，你应该使用 [`transform`](#method-transform) 方法。

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {#collection-method}

`mapWithKeys` 会迭代集合中的每一项，并且会将每一项的值传递到回调函数中。而回调函数应该返回一个关联数组来包含一个单一的键值对：

    $collection = collect([
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com'
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com'
        ]
    ]);

    $keyed = $collection->mapWithKeys(function ($item) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` {#collection-method}

`max` 方法返回给定键中最大的值：

    $max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-merge"></a>
#### `merge()` {#collection-method}

`merge` 方法会合并给定的数组到集合。数组中具有字符串作为键的值将会覆盖集合中相同键的值：

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

如果给定的数组的键是数值，则它的值将会被追加在集合的末尾：

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-min"></a>
#### `min()` {#collection-method}

`min`方法会返回集合中给定键的最小值：

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-only"></a>
#### `only()` {#collection-method}

`only` 方法返回集合中只包含选定的键的项：

    $collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

如果你想获得与 `only` 方法相反的结果，请使用 [except](#method-except) 方法。

<a name="method-pipe"></a>
#### `pipe()` {#collection-method}

`pipe` 方法可以传递给闭包当前集合的实例，并返回闭包执行的结果：

    $collection = collect([1, 2, 3]);

    $piped = $collection->pipe(function ($collection) {
        return $collection->sum();
    });

    // 6

<a name="method-pluck"></a>
#### `pluck()` {#collection-method}

`pluck` 方法将根据给定的键检索集合中所有项的值：

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $plucked = $collection->pluck('name');

    $plucked->all();

    // ['Desk', 'Chair']

你也可以指定将结果如何键化：

    $plucked = $collection->pluck('name', 'product_id');

    $plucked->all();

    // ['prod-100' => 'Desk', 'prod-200' => 'Chair']

<a name="method-pop"></a>
#### `pop()` {#collection-method}

`pop` 方法将从集合中剔除最后一个元素：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {#collection-method}

`prepend` 方法将在集合的起始端添加项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

你也可以传递第二个参数作为前置项目的键：

    $collection = collect(['one' => 1, 'two', => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two', => 2]

<a name="method-pull"></a>
#### `pull()` {#collection-method}

`pull` 方法从集合中返回给定键的项的同时将其从集合中剔除：

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {#collection-method}

`push` 方法将向集合中添加给定元素到集合的尾端：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {#collection-method}

`put` 方法在集合中设置给定的键和值：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {#collection-method}

`random` 方法随机的从集合中返回元素：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (retrieved randomly)

你也可以传递一个整型值到 `random`。如果整型值大于 `1`。则相应个数的随机项将会被返回：

    $random = $collection->random(3);

    $random->all();

    // [2, 4, 5] - (retrieved randomly)

<a name="method-reduce"></a>
#### `reduce()` {#collection-method}

`reduce` 方法将会缩小集合到单个值。它会通过迭代的方式将其值传递给随后的迭代器：

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    });

    // 6

上面的 `$carry` 在第一个迭代器中将会是 `null`；你也可以指定一个起始值到第二个参数：

    $collection->reduce(function ($carry, $item) {
        return $carry + $item;
    }, 4);

    // 10

<a name="method-reject"></a>
#### `reject()` {#collection-method}

`reject` 方法使用给定的回调从集合中返回过滤的值。你需要在回调中返回 `true` 来让其从结果中移除：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

如果需要与 `reject` 方法相反的结果，你可以使用 [`filter`](#method-filter) 方法。

<a name="method-reverse"></a>
#### `reverse()` {#collection-method}

`reverse` 方法会逆序排列集合中的项目：

    $collection = collect([1, 2, 3, 4, 5]);

    $reversed = $collection->reverse();

    $reversed->all();

    // [5, 4, 3, 2, 1]

<a name="method-search"></a>
#### `search()` {#collection-method}

`search` 方法根据给定的值搜索集合中的项，如果找到则返回该项的键，如果未找到则返回 `false`:

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

搜索使用的是疏松的比较，这意味着一个字符串和一个整数值可以被认为都是整数类型的值。如果需要进行严格比较，你应该传递 `true` 作为第二个参数：

    $collection->search('4', true);

    // false

另外，你也可以传递一个回调作为搜索的结果判断依据，在回调中首个返回真值的项目将会被检索到：

    $collection->search(function ($item, $key) {
        return $item > 5;
    });

    // 2

<a name="method-shift"></a>
#### `shift()` {#collection-method}

`shift` 方法会从集合中返回首个元素的同时从集合中剔除：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {#collection-method}

`shuffle` 方法随机打乱集合中项目的顺序：

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] // (generated randomly)

<a name="method-slice"></a>
#### `slice()` {#collection-method}

`slice` 方法根据给定的索引返回集合中的一小片：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

如果你想要限制返回的切片的大小，你可以传递期望的大小到第二个参数：

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

被返回的切片将会保留原集合中的键。如果不希望保存原集合中的键，那么你可以使用 `values` 方法来重置索引。

<a name="method-sort"></a>
#### `sort()` {#collection-method}

`sort` 方法将对集合进行排序。被排序的集合会保留原始的数组键。所以这个例子中我们将使用 [`values`](#method-values) 方法来重置键到连续的数字索引。

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

如果你的排序需要更多的逻辑支持，你可以传递一个回调到 `sort` 方法。你可以参考 PHP 文档的 [usort](http://php.net/manual/en/function.usort.php#refsect1-function.usort-parameters), 集合的 `sort` 方法就是在基于该方法的。

> {tip} 对于嵌套的数组或者对象的排序，请参照 [`sortBy`](#method-sortby) 和 [`sortByDesc`](#method-sortbydesc) 方法。

<a name="method-sortby"></a>
#### `sortBy()` {#collection-method}

`sortBy` 方法根据给定的键进行集合排序。所以这个例子中我们将使用 [`values`](#method-values) 方法来重置键到连续的数字索引。

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

你也可以传递你自己的回调来指导如何排序：

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function ($product, $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {#collection-method}

该方法与 [`sortBy`](#method-sortby) 方法有相同的运作方式，但是它是以集合中相反的顺序进行排序。

<a name="method-splice"></a>
#### `splice()` {#collection-method}

`splice` 方法根据指定的索引来从集合中剔除一片并返回这个切片：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

你也可以传递第二个参数到方法来限制切片的大小：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

另外，你可以传递第三个参数来作为从原集合中剔除元素的补偿：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-sum"></a>
#### `sum()` {#collection-method}

`sum` 方法返回集合中所有项的和：

    collect([1, 2, 3, 4, 5])->sum();

    // 15

如果集合中包含的是嵌套的数组或者对象。你应该传递键来指导如何进行求和：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

另外，你也可以传递一个回调来决定你所需要进行求和的值：

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function ($product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {#collection-method}

`take` 方法从集合中返回给定数量的项目：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

你也可以传递一个负值到方法，它将从集合的结尾开始返回：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-toarray"></a>
#### `toArray()` {#collection-method}

`toArray` 方法会将集合退化为原生的 PHP 数组。如果集合的值列是 [Eloquent](/docs/{{version}}/eloquent) 模型。模型也会被转化为数组：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> {note} `toArray` 也会转化所有的嵌套的对象到数组。如果你希望得到底层的原始数组，你可以使用 [`all`](#method-all) 方法。

<a name="method-tojson"></a>
#### `toJson()` {#collection-method}

`toJson` 方法转化集合到 JSON：

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {#collection-method}

`transform` 通过一个回调函数迭代处理集合中的项。集合中的项会在被迭代的结果所替换：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function ($item, $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> {note} 不像其他的集合方法，`transform` 会修改原始的集合自身。如果你希望创建一个新的集合来替代，请使用 [`map`](#method-map) 方法。

<a name="method-union"></a>
#### `union()` {#collection-method}

`union` 方法添加给定的数组到集合。如果给定的数组中包含的键在集合中已经存在，那么集合中相应键的项将会被保留，而不是被替换：

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['b']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], [3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {#collection-method}

`unique` 方法将返回所有在集合中具有独特性（去重）的项。返回的集合中保留了原始的数组键，所以这个例子中我们将使用 [`values`](#method-values) 方法来重置键到连续的数字索引：

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

当对嵌套的数组或者对象进行运算时，你应该指定一个键来作为唯一性的判定依据：

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

你也可以传递自己的回调到方法来进行判定：

    $unique = $collection->unique(function ($item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

<a name="method-values"></a>
#### `values()` {#collection-method}

`values` 方法返回一个键经过重置成连续整型索引的新集合：

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200]
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */
<a name="method-where"></a>
#### `where()` {#collection-method}

`where` 方法根据指定的键值对来进行集合的过滤：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
    */

`where` 方法使用疏松的比较模式。你可以使用 [`whereStrict`](#method-wherestrict) 方法来使用严格的比较模式。

<a name="method-wherestrict"></a>
#### `whereStrict()` {#collection-method}

该方法与 [`where`](#method-where) 具有相同的使用方法，但是它使用的是严格的比较模式。

<a name="method-wherein"></a>
#### `whereIn()` {#collection-method}

`whereIn` 方法根据给定的键值对中的值组来对集合中的项目进行匹配过滤：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
    [
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Desk', 'price' => 200],
    ]
    */

`whereIn` 方法使用的是严格的比较模式，如果你需要使用疏松的比较模式请使用 [`whereInLoose`](#method-whereinloose) 方法。

<a name="method-whereinloose"></a>
#### `whereInLoose()` {#collection-method}

该方法与 [`whereIn`](#method-wherein) 方法的运作相同，只是其使用的是疏松的比较模式。

<a name="method-zip"></a>
#### `zip()` {#collection-method}

`zip` 方法会根据相应的索引将给定的数组中的值与集合中项目的值进行合并：

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]
