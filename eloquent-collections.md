# Eloquent: Collections

- [Introduction](#introduction)
- [Available Methods](#available-methods)
- [Custom Collections](#custom-collections)

<a name="introduction"></a>
## Introduction

All multi-result sets returned by Eloquent are an instance of the `Illuminate\Database\Eloquent\Collection` object, including results retrieved via the `get` method or accessed via a relationship. The Eloquent collection object extends the Laravel [base collection](/docs/{{language}}/{{version}}/collections), so it naturally inherits dozens of methods used to fluently work with the underlying array of Eloquent models.

Of course, all collections also serve as iterators, allowing you to loop over them as if they were simple PHP arrays:

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

However, collections are much more powerful than arrays and expose a variety of map / reduce operations that may be chained using an intuitive interface. For example, let's remove all inactive models and gather the first name for each remaining user:

    $users = App\User::where('active', 1)->get();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> **Note:** While most Eloquent collection methods return a new instance of an Eloquent collection, the `pluck`, `keys`, `zip`, `collapse`, `flatten` and `flip` methods return a [base collection](/docs/{{language}}/{{version}}/collections) instance.

<a name="available-methods"></a>
## Available Methods

### The Base Collection

All Eloquent collections extend the base [Laravel collection](/docs/{{language}}/{{version}}/collections) object; therefore, they inherit all of the powerful methods provided by the base collection class:

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

[all](/docs/{{language}}/{{version}}/collections#method-all)
[chunk](/docs/{{language}}/{{version}}/collections#method-chunk)
[collapse](/docs/{{language}}/{{version}}/collections#method-collapse)
[contains](/docs/{{language}}/{{version}}/collections#method-contains)
[count](/docs/{{language}}/{{version}}/collections#method-count)
[diff](/docs/{{language}}/{{version}}/collections#method-diff)
[each](/docs/{{language}}/{{version}}/collections#method-each)
[every](/docs/{{language}}/{{version}}/collections#method-every)
[filter](/docs/{{language}}/{{version}}/collections#method-filter)
[first](/docs/{{language}}/{{version}}/collections#method-first)
[flatten](/docs/{{language}}/{{version}}/collections#method-flatten)
[flip](/docs/{{language}}/{{version}}/collections#method-flip)
[forget](/docs/{{language}}/{{version}}/collections#method-forget)
[forPage](/docs/{{language}}/{{version}}/collections#method-forpage)
[get](/docs/{{language}}/{{version}}/collections#method-get)
[groupBy](/docs/{{language}}/{{version}}/collections#method-groupby)
[has](/docs/{{language}}/{{version}}/collections#method-has)
[implode](/docs/{{language}}/{{version}}/collections#method-implode)
[intersect](/docs/{{language}}/{{version}}/collections#method-intersect)
[isEmpty](/docs/{{language}}/{{version}}/collections#method-isempty)
[keyBy](/docs/{{language}}/{{version}}/collections#method-keyby)
[keys](/docs/{{language}}/{{version}}/collections#method-keys)
[last](/docs/{{language}}/{{version}}/collections#method-last)
[map](/docs/{{language}}/{{version}}/collections#method-map)
[merge](/docs/{{language}}/{{version}}/collections#method-merge)
[pluck](/docs/{{language}}/{{version}}/collections#method-pluck)
[pop](/docs/{{language}}/{{version}}/collections#method-pop)
[prepend](/docs/{{language}}/{{version}}/collections#method-prepend)
[pull](/docs/{{language}}/{{version}}/collections#method-pull)
[push](/docs/{{language}}/{{version}}/collections#method-push)
[put](/docs/{{language}}/{{version}}/collections#method-put)
[random](/docs/{{language}}/{{version}}/collections#method-random)
[reduce](/docs/{{language}}/{{version}}/collections#method-reduce)
[reject](/docs/{{language}}/{{version}}/collections#method-reject)
[reverse](/docs/{{language}}/{{version}}/collections#method-reverse)
[search](/docs/{{language}}/{{version}}/collections#method-search)
[shift](/docs/{{language}}/{{version}}/collections#method-shift)
[shuffle](/docs/{{language}}/{{version}}/collections#method-shuffle)
[slice](/docs/{{language}}/{{version}}/collections#method-slice)
[sort](/docs/{{language}}/{{version}}/collections#method-sort)
[sortBy](/docs/{{language}}/{{version}}/collections#method-sortby)
[sortByDesc](/docs/{{language}}/{{version}}/collections#method-sortbydesc)
[splice](/docs/{{language}}/{{version}}/collections#method-splice)
[sum](/docs/{{language}}/{{version}}/collections#method-sum)
[take](/docs/{{language}}/{{version}}/collections#method-take)
[toArray](/docs/{{language}}/{{version}}/collections#method-toarray)
[toJson](/docs/{{language}}/{{version}}/collections#method-tojson)
[transform](/docs/{{language}}/{{version}}/collections#method-transform)
[unique](/docs/{{language}}/{{version}}/collections#method-unique)
[values](/docs/{{language}}/{{version}}/collections#method-values)
[where](/docs/{{language}}/{{version}}/collections#method-where)
[whereLoose](/docs/{{language}}/{{version}}/collections#method-whereloose)
[zip](/docs/{{language}}/{{version}}/collections#method-zip)

</div>

<a name="custom-collections"></a>
## Custom Collections

If you need to use a custom `Collection` object with your own extension methods, you may override the `newCollection` method on your model:

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

Once you have defined a `newCollection` method, you will receive an instance of your custom collection anytime Eloquent returns a `Collection` instance of that model. If you would like to use a custom collection for every model in your application, you should override the `newCollection` method on a model base class that is extended by all of your models.
