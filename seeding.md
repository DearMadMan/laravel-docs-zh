# 数据库: 数据填充（Seeding)

- [前言](#introduction)
- [编写 Seeders](#writing-seeders)
    - [使用模型工厂](#using-model-factories)
    - [调用其它 Seeders](#calling-additional-seeders)
- [执行 Seeders](#running-seeders)

<a name="introduction"></a>
## 前言

Laravel 使用 seed 类提供一种简便的方法填充你用于测试的数据。所有的 seed 类文件都被存储在 `database/seeds` 目录下。Seed 类可以拥有任意的类名，但是你应该遵循某些命名规范，比如 `UsersTableSeeder`，等等。默认的 Laravel 为你提供了 `DatabaseSeeder` 类，在这个类中，你可以使用 `call` 方法来调用其他的 seed 类，这为你顺序的进行数据填充提供了便利。

<a name="writing-seeders"></a>
## 编写 Seeders

你可以使用 `make:seeder` [Artisan 命令](/docs/{{version}}/artisan) 来生成一个 seeder。所有通过框架生成的 seeders 都会被放在 `database/seeds` 目录下：

    php artisan make:seeder UsersTableSeeder

seeder 类中默认的只包含了一个方法：`run`。该方法会在 `db:seed` [Artisan command](/docs/{{version}}/artisan) 执行时被调用。在 `run` 方法中，你可以填充任意你想填充的数据到数据库中，你可以使用 [查询生成器](/docs/{{version}}/queries) 手动的插入数据，或者你也可以使用 [Eloquent 模型工厂](/docs/{{version}}/database-testing#model-factories)。

让我们来做个示例，我们修改 Laravel 默认提供的 `DatabaseSeeder` 类，让我们在 `run` 方法中进行一些数据插入：

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Eloquent\Model;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeds.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### 使用模型工厂

当然，手动的为每个模型 seed 指定属性是一件非常麻烦的事情。你可以使用 [模型工厂](/docs/{{version}}/database-testing#model-factories) 来方便的生成大量的数据记录。首先，你需要查询一下 [模型工厂的文档](/docs/{{version}}/database-testing#model-factories) 来学习一下如何定义你的模型工厂。一旦你定义了模型工厂，你就可以使用 `factory` 帮助方法将记录插入到数据库中。

比如，让我们创建 50 个用户并且为每个用户附加一个关系：

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function($u) {
            $u->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### 调用其他 Seeders

在 `DatabaseSeeder` 类中，你可以使用 `call` 方法来执行其他的 seed 类。使用 `call` 方法允许你将数据库填充分解到多个文件中，这样不会使一个单独的 seed 文件过于庞大。你只需要简单的传递 seeder 的类名到方法中就可以进行调用：

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        $this->call(UsersTableSeeder::class);
        $this->call(PostsTableSeeder::class);
        $this->call(CommentsTableSeeder::class);
    }

<a name="running-seeders"></a>
## 执行 Seeders

一旦你编写完成 seeder 类，你就可以使用 `db:seed` Artisan 命令来将数据填充到数据库中。默认的，`db:seed` 命令会直接执行 `DatabaseSeeder` 类，你也可以在这个类中调用其他的 seed 类。事实上，你也可以使用 `--class` 选项来单独的指定需要填充的类：

    php artisan db:seed

    php artisan db:seed --class=UsersTableSeeder

你也可以在使用 `migrate:refresh` 命令时运行 seed，这将回滚所有的迁移，然后再执行迁移，之后进行填充数据。这条命令对于应用的数据库重置非常有用：

    php artisan migrate:refresh --seed
