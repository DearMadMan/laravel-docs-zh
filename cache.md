# 缓存

- [配置](#configuration)
    - [驱动需求](#driver-prerequisites)
- [缓存用法](#cache-usage)
    - [获取缓存实例](#obtaining-a-cache-instance)
    - [检索缓存项](#retrieving-items-from-the-cache)
    - [新增缓存项](#storing-items-in-the-cache)
    - [移除缓存项](#removing-items-from-the-cache)
- [缓存标签](#cache-tags)
    - [存储已标记缓存项](#storing-tagged-cache-items)
    - [访问已标记缓存项](#accessing-tagged-cache-items)
    - [删除已标记缓存项](#removing-tagged-cache-items)
- [添加自定义缓存驱动](#adding-custom-cache-drivers)
    - [编写驱动](#writing-the-driver)
    - [注册驱动](#registering-the-driver)
- [事件](#events)

<a name="configuration"></a>
## 配置

Laravel 对多种缓存系统提供了统一的 API。缓存的配置文件存放在 `config/cache.php`。你可以在这个文件中指定整个应用默认使用何种缓存驱动。Laravel 支持当前主流的缓存系统如 [Memcached](http://memcached.org) and [Redis](http://redis.io)。

缓存的配置文件也包含了一些额外的配置选项，这些选项在文件中都有文档注释，你应该确保自己已经读了这些选项注释。默认的，Laravel 配置使用 `file` 缓存驱动，该驱动会在文件系统中存储序列化的缓存对象。对于大型应用，建议使用内存级的缓存，如 Memcached 或者 Redis。你甚至可以在 Laravel 中配置多种缓存配置到相同的驱动。

<a name="driver-prerequisites"></a>
### 驱动需求

#### 数据库

当使用 `database` 缓存驱动时，你需要建立一个表来包含这些缓存项。你可以根据下面的 `Schema` 定义来建立表文件：

    Schema::create('cache', function($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} 你也可以通过使用 `php artisan cache:table` Artisan 命令来生成正确的缓存表结构迁移。

#### Memcached

使用 Memcached 缓存需要安装 [Memcached PECL package](http://pecl.php.net/package/memcached)。你可以在 `config/cache.php` 配置文件中列出你所有的 Memcached 服务:

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

你也可以使用 UNIX socket 路径来设置 `host`。如果你这么做，你需要设置 `port` 为 `0`:

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

在你使用 Redis 缓存之前，你需要先通过 Composer 安装 `predis/predis`。

关于更多的 Redis 配置信息，你可以参考 [Laravel documentation page](/docs/{{language}}/{{version}}/redis#configuration)。

<a name="cache-usage"></a>
## 缓存用法

<a name="obtaining-a-cache-instance"></a>
### 获取缓存实例

`Illuminate\Contracts\Cache\Factory` 和 `Illuminate\Contracts\Cache\Repository` [契约](/docs/{{language}}/{{version}}/contracts) 用于提供对 Laravel 缓存服务的访问。`Factory` 契约提供了应用中所定义的缓存驱动的访问权限。`Repository` 契约通常是一个基于你的 `cache` 配置文件所使用的默认缓存驱动的实现。

事实上，你也可以使用 `Cache` 假面，在这篇文档中，我们都是使用 `Cache` 假面进行举例。`Cache` 假面提供了一种方便简洁的方式来访问 Laravel 底层缓存契约的实现。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

            //
        }
    }

#### 访问多种缓存存储

你可以通过 `Cache` 假面的 `store` 方法来访问多种缓存存储。传递到 `store` 方法的 key 应该与你的 `cache` 配置文件中的 `stores` 配置项的列表之一相匹配：

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### 获取缓存项

你可以通过使用 `Cache` 假面的 `get` 方法来从缓存中获取相关项的值。如果该项在缓存中并不存在，则返回 `null` 。如果你需要，你也可以传递第二个参数到 `get` 方法，这个参数所传递的值会在缓存中项不存在时被返回:

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');


你甚至可以传递 `Closure` 作为默认值。如果缓存的项不存在，`Closure` 所返回的值将被做为默认值。传递闭包的方式可以使你从数据库或者其他外部服务中延迟获取默认值：

    $value = Cache::get('key', function() {
        return DB::table(...)->get();
    });

#### 检查缓存项是否存在

你可以使用 `has` 方法来检查缓存中是否存在该项：

    if (Cache::has('key')) {
        //
    }

#### 递增 / 递减 Values

你可以使用 `increment` 和 `decrement` 方法来调整缓存项目中的整型值。这两个方法都可以接受一个可选的第二个参数来进行相应的数值调整：

    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

#### 检索 & 存储

有时候，你可能希望从缓存中检索出一个项目，但是当该项不存在的时候，你也想存储一个默认值到该项。比如，你希望从缓存中检索出所有用户。但是它们并不存在，所以你需要从数据库中获取到所有用户，然后添加到缓存中。你可以使用 `Cache::remember` 方法来做到这些：

    $value = Cache::remember('users', $minutes, function() {
        return DB::table('users')->get();
    });

如果缓存中没有检索到该项，传递到 `remeber` 方法中的闭包将会被执行并且其执行结果将会存储到缓存中。

#### 检索 & 删除

如果你需要检索一个项目，并在检索到的同时从缓存中删除该项，你可以使用 `pull` 方法。就像 `get` 方法一样，如果未检索到该项，将会返回 `null` :

    $value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### 新增缓存项

你可以使用 `Cache` 假面的 `put` 方法来存储项目到缓存中。当你存储一个项到缓存中时，你需要指定一个该项需要被缓存的分钟值:

    Cache::put('key', 'value', $minutes);

除了传递一个数值作为缓存过期的分钟值，你也可以通过传递一个 PHP `DateTime` 实例来设置缓存的失效时间：

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### 如果不存在则存储

`add` 方法只会在相应的项在缓存中不存在时才会被添加进缓存。该方法会在项目已经在缓存中时返回 `true`。否则返回 `false`：

    Cache::add('key', 'value', $minutes);

#### 永久存储缓存项

`forever` 方法可以用来将项目永久的添加进缓存。由于这些项并没有被设置过期时间，所以只有手动的使用 `forget` 方法才能移除这些缓存项：

    Cache::forever('key', 'value');

> {tip} 如果你使用 Memcached 驱动，被永久存储的缓存项可能会在其缓存达到限制大小时被清除掉。

<a name="removing-items-from-the-cache"></a>
### 删除缓存项

你可以使用 `Cache` 假面的 `forget` 方法来从缓存中移除某项：

    Cache::forget('key');

你可以使用 `flush` 方法来擦除所有的缓存：

    Cache::flush();

> {note} 擦除缓存并不会根据前缀来进行智能擦除，它会移除所有的缓存。所以如果你的应用和其他的应用共享缓存，你应该谨慎的使用该方法。

<a name="cache-tags"></a>
## 缓存标签

> {note} 缓存标签并不支持 `file` 或者 `database` 缓存驱动。另外，对于支持将多种标签标记为永久存储的驱动，性能最好的是能够提供自动清除过期记录的驱动，比如 `memcached`。

<a name="storing-tagged-cache-items"></a>
### 存储已标记缓存项

缓存标签允许你将相关的项目进行关联标记。并且允许一次性清除所有给定标签的缓存项。你可以通过有序的标签数组来访问被标记的缓存项目。比如，让我们访问被标记的项目并使用 `put` 方法来设置缓存值：

	Cache::tags(['people', 'artists'])->put('John', $john, $minutes);

	Cache::tags(['people', 'authors'])->put('Anne', $anne, $minutes);

<a name="accessing-tagged-cache-items"></a>
### 访问已标记缓存项

为了访问被标记了的缓存项，你需要传递相应的有序标签列表到 `tags` 方法，然后调用 `get` 方法并传递你所需要检索的键：

	$john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### 删除已标记缓存项

你可以一次性的擦除分配的标记或者标记列表中的所有项。比如，下面的方法将会删除所有被标记为 `people` 或者 `authors` 的缓存项，也会删除两者组成的有序列标签里的所有缓存项。所以，`Anne` 和 `John` 都会被从缓存中移除：

	Cache::tags(['people', 'authors'])->flush();

下面的语句将会作为上面语句的对比，将只会从缓存中删除 `authors` 标签的项目，所以 `Anne` 会被删除，而 `John` 将被保留：

	Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## 添加自定义缓存驱动

<a name="writing-the-driver"></a>
### 编写驱动

为了编写自定义的缓存驱动，我们首先需要实现 `Illuminate\Contracts\Cache\Store` [契约](/docs/{{language}}/{{version}}/contracts)。所以，一个 MongoDB 缓存的实现应该看起来像这样：

    <?php

    namespace App\Extensions;

    use Illuminate\Contracts\Cache\Store;

    class MongoStore implements Store
    {
        public function get($key) {}
        public function many(array $keys);
        public function put($key, $value, $minutes) {}
        public function putMany(array $values, $minutes);
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}
        public function getPrefix() {}
    }

我们只需要使用 MongoDB 连接实现上面每个方法就可以了。你可以看一下框架中的 `Illuminate\Cache\MemcachedStore` 源码来了解怎么实现这些方法。当我们这些方法的实现完成之后，我们就可以完成自定义驱动的注册了。

    Cache::extend('mongo', function($app) {
        return Cache::repository(new MongoStore);
    });

> {tip} 如果你在为自定义的缓存文件应该存放在哪里而感到疑惑，你可以在 `app` 目录中创建一个 `Extensions` 命名空间。事实上，你应该谨记，Laravel 并不死板的限制你的目录结构，你应该可以根据自己的习惯自由的管理你的应用目录结构。

<a name="registering-the-driver"></a>
### 注册驱动

为了在 Laravel 中注册自定义的驱动，我们需要使用 `Cache` 假面的 `extend` 方法。你可以在 `App\Providers\AppServiceProvider` 中的 `boot` 方法中调用 `Cache::extend` 方法或者你可以自己创建一个提供者来专门做这件事，但是请不要忘记在你的 `config/app.php` 配置文件中注册这个提供者：

    <?php

    namespace App\Providers;

    use App\Extensions\MongoStore;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\ServiceProvider;

    class CacheServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Cache::extend('mongo', function($app) {
                return Cache::repository(new MongoStore);
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

第一个被传递到 `extend` 方法中的参数应该是驱动的名称。这个名称应该和你的 `config/cache.php` 配置文件中的 `driver` 选项一致。而第二个参数是一个闭包，该闭包应该返回一个 `Illuminate\Cache\Repository` 的实现。在闭包中将会被传递一个 `$app` 实例，这个实例是 Laravel 中的 [服务容器](/docs/{{language}}/{{version}}/container) 的实例。

当扩展被注册之后，你就可以在 `config/cache.php` 配置文件中更新驱动选型 `driver` 为你的扩展的名称。

<a name="events"></a>
## 事件

如果你想在任何缓存被操作时执行额外的代码，你可能需要监听缓存的触发 [事件](/docs/{{language}}/{{version}}/events)。通常的你应该存放这些事件监听器到你的 `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Cache\Events\CacheHit' => [
            'App\Listeners\LogCacheHit',
        ],

        'Illuminate\Cache\Events\CacheMissed' => [
            'App\Listeners\LogCacheMissed',
        ],

        'Illuminate\Cache\Events\KeyForgotten' => [
            'App\Listeners\LogKeyForgotten',
        ],

        'Illuminate\Cache\Events\KeyWritten' => [
            'App\Listeners\LogKeyWritten',
        ],
    ];
