# 数据库测试

- [前言](#introduction)
- [每次测试完成后重置数据库](#resetting-the-database-after-each-test)
    - [使用迁移](#using-migrations)
    - [使用事务](#using-transactions)
- [编写工厂](#writing-factories)
    - [工厂类型](#factory-types)
- [使用工厂](#using-factories)
    - [创建模型](#creating-models)
    - [持久化模型](#persisting-models)
    - [关联](#relationships)

<a name="introduction"></a>
## 前言

Laravel 也提供了各种有用的工具来使测试基于数据库驱动的应用更为简单。首先，你可以使用 `seeInDatabase` 方法来断言给定的规范是否能在数据库中匹配到相应的数据。比如，如果你希望验证 `users` 表中是否含有 `email` 为 `sally@example.com` 的记录，那么你可以这么做：

    public function testDatabase()
    {
        // Make call to application...

        $this->seeInDatabase('users', [
            'email' => 'sally@example.com'
        ]);
    }

当然，类似 `seeInDatabase` 等方法仅仅是为了测试的方便。你可以自由的使用任意的 PHPUnit 自带的断言方法来补充你的测试。

<a name="resetting-the-database-after-each-test"></a>
## 在每个测试后重置数据库

我们通常要在每次测试后还原数据库，以便之前的测试数据不会对随后的测试产生干扰。

<a name="using-migrations"></a>
### 使用迁移

一种选择是在下个测试之前进行数据库回滚和迁移操作。Laravel 提供了简洁的 `DatabaseMigrations` trait 来自动的处理这些，你只需要在测试类引入该性状就可以了：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseMigrations;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }

<a name="using-transactions"></a>
### 使用事务

另外一种选择就是在每个测试用例中都使用数据库的事务进行包装，这一次，Laravel 提供了方便的 `DatabaseTransactions` trait 来自动的处理这些：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use DatabaseTransactions;

        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }

> {note} 这一性状只会包装默认数据库连接的事务。如果你的应用使用多个数据库连接，那么你需要手动的为这些连接处理事务逻辑。

<a name="model-factories"></a>
## 编写工厂

当测试时，通常需要在执行测试之前在数据库中添加一些测试所需要的数据记录。Laravel 允许你使用 "factories" 来对你的 [Eloquent 模型](/{{language}}/{{version}}/eloquent) 定义一些默认的属性设置来取代这些手动的添加数据的行为。在开始之前，我们来看一下应用的 `database/factories/ModelFactory.php` 文件。该文件中只包含了一个工厂的定义：

    $factory->define(App\User::class, function (Faker\Generator $faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => bcrypt(str_random(10)),
            'remember_token' => str_random(10),
        ];
    });

在工厂定义中的闭包中，你可以返回一些模型中的用作测试的默认值。这个闭包会接收一个 [Faker](https://github.com/fzaninotto/Faker) PHP 类库的实例，它会帮你生成一些随机数据用于测试。

当然，你可以自由的在 `ModelFactory.php` 文件中添加额外的工厂。你也可以创建额外的工厂文件来有效的组织管理。比如，你可以在 `database/factories` 目录下创建一个 `UserFactory.php` 和一个 `CommentFactory.php` 文件。所有存放在 `factories` 目录下的工厂文件都会自动的被 Laravel 加载。

<a name="factory-types"></a>
### 工厂类型

有时候，你可能希望在同一个 Eloquent 模型上有多个工厂类型。比如，可能你希望除了普通用户之外还有一个管理员的工厂。你可以使用 `defineAs` 方法来定义这些工厂：

    $factory->defineAs(App\User::class, 'admin', function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
            'admin' => true,
        ];
    });

你可以使用 `raw` 方法来检索基础工厂的属性，这样可用于属性的复用，然后合并添加你所需要的额外属性：

    $factory->defineAs(App\User::class, 'admin', function ($faker) use ($factory) {
        $user = $factory->raw(App\User::class);

        return array_merge($user, ['admin' => true]);
    });

<a name="using-factories"></a>
## 使用工厂

<a name="creating-models"></a>
### 创建模型

一旦你定义了工厂，你就可以在你的测试文件或者数据库 seed 文件中使用全局帮助函数 `factory` 工厂方法来生成模型的实例。那么，让我们来看一些创建模型的示例。首先，我们将使用 `make` 方法，它将会创建一些模型但是却不会将其保存到数据库：

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // Use model in tests...
    }

你也可以根据给定的类型创建多个模型的集合：

    // Create three App\User instances...
    $users = factory(App\User::class, 3)->make();

    // Create an "admin" App\User instance...
    $user = factory(App\User::class, 'admin')->make();

    // Create three "admin" App\User instances...
    $users = factory(App\User::class, 'admin', 3)->make();

#### 覆盖属性

你可以通过传递一个数组到 `make` 方法中来覆盖模型中的默认值。只有指定的部分会被替换掉，而其它部分都会被保留为工厂定义的默认值：

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);

<a name="persisting-models"></a>
### 持久化模型

`create` 方法不仅仅会创建一个模型实例，它还会使用 Eloquent 的 `save` 方法将其存储到数据库中：

    public function testDatabase()
    {
        // Create a single App\User instance...
        $user = factory(App\User::class)->create();

        // Create three App\User instances...
        $users = factory(App\User::class, 3)->create();

        // Use model in tests...
    }

你也可以传递一个数组到 `create` 方法中来覆盖默认的属性值：

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
    ]);

<a name="relationships"></a>
### 关联

在这个例子中，我们甚至会为创建的模型附加一个关联模型。当使用 `create` 方法来创建多个模型时，会返回一个 [集合实例](/{{language}}/{{version}}/eloquent-collections)，这允许你使用集合实例的方法，比如 `each` :

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function ($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

#### 关联 & 属性闭包

你也可以在工厂定义内使用一个闭包来附加关系模型，比如，如果你希望创建一个 `Post` 时也创建一个关联的 `User` 实例，你可以这么做：

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            }
        ];
    });

这些闭包还会接收工厂所包含的当前属性数组：

    $factory->define(App\Post::class, function ($faker) {
        return [
            'title' => $faker->title,
            'content' => $faker->paragraph,
            'user_id' => function () {
                return factory(App\User::class)->create()->id;
            },
            'user_type' => function (array $post) {
                return App\User::find($post['user_id'])->type;
            }
        ];
    });
