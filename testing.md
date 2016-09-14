# 测试

- [前言](#introduction)
- [环境](#environment)
- [创建 & 运行测试](#creating-and-running-tests)

<a name="introduction"></a>
## 前言

Laravel 的构建一直是测试优先的。事实上，Laravel 支持 PHPUnit 测试，你的应用的根目录已经包含了 `phpunit.xml` 文件。同时，Laravel 也附带了一些方便的帮助方法可以使你丰满应用的测试。

在 `tests` 目录中提供了一个 `ExampleTest.php` 文件。在安装完成 Laravel 应用之后，你只需要在根目录运行 `phpunit` 命令就可以执行测试。

<a name="environment"></a>
## 环境

当运行测试时，Laravel 会自动的设置配置环境为 `testing`。同时，Laravel 会自动的配置 session 和 缓存为 `array` 驱动。这意味着会话或者缓存数据在测试期间不会被持久化。

如果你需要，你完全可以自己创建一个测试环境。`testing` 环境变量是在 `phpunit.xml` 文件中被配置的，但是你要确保在运行测试之前先运行 `config:clear` Artisan 命令来清除配置缓存。

<a name="creating-and-running-tests"></a>
## 创建 & 允许测试

你可以使用 `make:test` Artisan 命令来创建一个测试用例：

    php artisan make:test UserTest

这个命令会在 `tests` 目录下创建一个新的 `UserTest` 类。你可以像使用 PHPUnit 一样接着在类中定义一些测试方法。然后你只需要在终端执行 `phpunit` 命令就可以运行测试：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class UserTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testExample()
        {
            $this->assertTrue(true);
        }
    }

> {note} 如果你在测试用例中定义了自己的 `setUp` 方法，你需要确保优先调用 `parent::setUp` 方法。
