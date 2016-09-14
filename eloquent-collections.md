# Eloquent: 集合

- [前言](#introduction)
- [可用的方法](#available-methods)
- [自定义集合](#custom-collections)

<a name="introduction"></a>
## 前言

通过 Eloquent 返回的多结果集是一个 `Illuminate\Database\Eloquent\Collection` 对象，这包括通过使用 `get` 方法或者通过访问关联模型所获得的结果。Eloquent 的集合对象继承自 Laravel 的 [基础集合](/docs/{{version}}/collections)，所以它自然地继承了几十种流畅执行的方法来与 Eloquent 模型的底层数组进行交互。

当然，所有的集合都可作为迭代器使用，这允许你像普通数组一样对他们进行循环操作：

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

事实上，集合要比数组强大的多，它提供了多种强大的接口方法可以直观的链式调用。比如，让我们删除所有未激活的用户，并且收集所有保留用户的首个名字：

    $users = App\User::where('active', 1)->get();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} 大多数的 Eloquent 集合方法都会返回一个新的 Eloquent 集合的实例，而 `pluck`，`keys`，`zip`，`collapse`，`flatten` 和 `flip` 方法返回 [基础集合](/docs/{{version}}/collections) 的实例。同样的，如果 `map` 方法所返回的结果中并没有包含任何的 Eloquent 模型，那么它将会自动的转换为基础集合。

<a name="available-methods"></a>
## 可用的方法

### 基础集合

所有的 Eloquent 集合都继承自 [Laravel Collection](/docs/{{version}}/collections) 对象。因此，所有继承自基础集合类的强大方法都是可用的:

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

[all](/docs/{{version}}/collections#method-all)
[avg](/docs/{{version}}/collections#method-avg)
[chunk](/docs/{{version}}/collections#method-chunk)
[collapse](/docs/{{version}}/collections#method-collapse)
[combine](/docs/{{version}}/collections#method-combine)
[contains](/docs/{{version}}/collections#method-contains)
[count](/docs/{{version}}/collections#method-count)
[diff](/docs/{{version}}/collections#method-diff)
[diffKeys](/docs/{{version}}/collections#method-diffkeys)
[each](/docs/{{version}}/collections#method-each)
[every](/docs/{{version}}/collections#method-every)
[except](/docs/{{version}}/collections#method-except)
[filter](/docs/{{version}}/collections#method-filter)
[first](/docs/{{version}}/collections#method-first)
[flatMap](/docs/{{version}}/collections#method-flatmap)
[flatten](/docs/{{version}}/collections#method-flatten)
[flip](/docs/{{version}}/collections#method-flip)
[forget](/docs/{{version}}/collections#method-forget)
[forPage](/docs/{{version}}/collections#method-forpage)
[get](/docs/{{version}}/collections#method-get)
[groupBy](/docs/{{version}}/collections#method-groupby)
[has](/docs/{{version}}/collections#method-has)
[implode](/docs/{{version}}/collections#method-implode)
[intersect](/docs/{{version}}/collections#method-intersect)
[isEmpty](/docs/{{version}}/collections#method-isempty)
[keyBy](/docs/{{version}}/collections#method-keyby)
[keys](/docs/{{version}}/collections#method-keys)
[last](/docs/{{version}}/collections#method-last)
[map](/docs/{{version}}/collections#method-map)
[max](/docs/{{version}}/collections#method-max)
[merge](/docs/{{version}}/collections#method-merge)
[min](/docs/{{version}}/collections#method-min)
[only](/docs/{{version}}/collections#method-only)
[pluck](/docs/{{version}}/collections#method-pluck)
[pop](/docs/{{version}}/collections#method-pop)
[prepend](/docs/{{version}}/collections#method-prepend)
[pull](/docs/{{version}}/collections#method-pull)
[push](/docs/{{version}}/collections#method-push)
[put](/docs/{{version}}/collections#method-put)
[random](/docs/{{version}}/collections#method-random)
[reduce](/docs/{{version}}/collections#method-reduce)
[reject](/docs/{{version}}/collections#method-reject)
[reverse](/docs/{{version}}/collections#method-reverse)
[search](/docs/{{version}}/collections#method-search)
[shift](/docs/{{version}}/collections#method-shift)
[shuffle](/docs/{{version}}/collections#method-shuffle)
[slice](/docs/{{version}}/collections#method-slice)
[sort](/docs/{{version}}/collections#method-sort)
[sortBy](/docs/{{version}}/collections#method-sortby)
[sortByDesc](/docs/{{version}}/collections#method-sortbydesc)
[splice](/docs/{{version}}/collections#method-splice)
[sum](/docs/{{version}}/collections#method-sum)
[take](/docs/{{version}}/collections#method-take)
[toArray](/docs/{{version}}/collections#method-toarray)
[toJson](/docs/{{version}}/collections#method-tojson)
[transform](/docs/{{version}}/collections#method-transform)
[union](/docs/{{version}}/collections#method-union)
[unique](/docs/{{version}}/collections#method-unique)
[values](/docs/{{version}}/collections#method-values)
[where](/docs/{{version}}/collections#method-where)
[whereStrict](/docs/{{version}}/collections#method-wherestrict)
[whereIn](/docs/{{version}}/collections#method-wherein)
[whereInLoose](/docs/{{version}}/collections#method-whereinloose)
[zip](/docs/{{version}}/collections#method-zip)

</div>

<a name="custom-collections"></a>
## 自定义集合

如果你需要使用自定义的 `Collection` 对象和自己的扩展方法，那么你需要在你的模型中复写 `newCollection` 方法：

    <?php

    namespace App;

    use App\CustomCollection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Create a new Eloquent Collection instance.
         *
         * @param  array  $models
         * @return \Illuminate\Database\Eloquent\Collection
         */
        public function newCollection(array $models = [])
        {
            return new CustomCollection($models);
        }
    }

一旦你定义完成 `newCollection` 方法，那么无论什么时候，Eloquent 返回的模型的 `Collection` 实例都会是你所自定义的集合。如果你想要应用中所有的模型都使用自定义的集合，那么你需要在所有模型所继承的基类中复写 `newCollection` 方法。
