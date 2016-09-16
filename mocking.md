# 模拟

- [前言](#introduction)
- [事件](#mocking-events)
- [任务](#mocking-jobs)
- [假面](#mocking-facades)

<a name="introduction"></a>
## 前言

当测试 Laravel 应用时，你可能会希望在进行测试时并不真正的执行某些操作，而只是模仿应用中的某些方面。比如，当测试一个可以触发事件的控制器时，你可能希望去模拟事件监听器，而不是在测试时真实的执行事件。这允许你在测试控制器的 HTTP 响应时完全不需要考虑事件监听器的执行，因为事件监听器可以在它们自己的测试用例里进行测试。

Laravel 为模拟事件，任务和假面提供了及其有用的帮助方法。这些帮助方法主要用于提供便利的应用层模拟，所以你并不需要手动的构造复杂的模拟方法的调用。当然，你可以自由的使用 [Mockery](http://docs.mockery.io/en/latest/) 或者 PHPUnit 来创建自己的模拟或者侦探。

<a name="mocking-events"></a>
## 事件

如果你在 Laravel 中大量使用了事件系统，那么你可能会想在进行测试时不触发事件或者模拟运行一些事件。比如，如果你测试用户注册，你可能并不想触发所有 `UserRegistered` 事件，因为这可能会发送电子邮件等。

Laravel 提供了方便的 `expectsEvents` 方法来验证预期的事件是否触发，但是会避免这些事件的真正的执行：

    <?php

    use App\Events\UserRegistered;

    class ExampleTest extends TestCase
    {
        /**
         * Test new user registration.
         */
        public function testUserRegistration()
        {
            $this->expectsEvents(UserRegistered::class);

            // Test user registration...
        }
    }

你也可以使用 `doesntExpectEvents` 方法来验证给定的事件没有被触发：

    <?php

    use App\Events\OrderShipped;
    use App\Events\OrderFailedToShip;

    class ExampleTest extends TestCase
    {
        /**
         * Test order shipping.
         */
        public function testOrderShipping()
        {
            $this->expectsEvents(OrderShipped::class);
            $this->doesntExpectEvents(OrderFailedToShip::class);

            // Test order shipping...
        }
    }

如果你需要避免所有的事件触发，你可以使用 `withoutEvents` 方法，当这个方法被调用时，所有事件的监听器都将会被模拟：

    <?php

    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->withoutEvents();

            // Test user registration code...
        }
    }

<a name="mocking-jobs"></a>
## 任务

有时候，你想在构建请求到应用时，简单的测试一下控制器中是否进行了指定任务的分发。这可以使你隔离测试你的路由 / 控制器和你的任务逻辑，当然，你可以再写一个测试用例来单独的测试任务的执行。

Laravel 提供了方便的 `expectsJobs` 方法来验证期望的任务是否被分发，但是任务并不会被真正的分发执行：

    <?php

    class App\Jobs\ShipOrder;

    class ExampleTest extends TestCase
    {
        public function testOrderShipping()
        {
            $this->expectsJobs(ShipOrder::class);

            // Test order shipping...
        }
    }

> {note} 这个方法仅用来测试通过 `DispatchesJobs` trait 或者 `dispatch` 帮助方法所进行的分发。它并不检测直接使用 `Queue::push` 发布的任务。

类似事件模拟，你也可以使用 `doesntExpectJobs` 方法来断言任务并没有被分发：

    <?php

    class App\Jobs\ShipOrder;

    class ExampleTest extends TestCase
    {
        /**
         * Test order cancellation.
         */
        public function testOrderCancellation()
        {
            $this->doesntExpectJobs(ShipOrder::class);

            // Test order cancellation...
        }
    }

另外，你也可以使用 `withoutJobs` 方法来忽略所有任务的分发。当在测试方法中使用这个方法时，所有的通过测试进行的分发任务都将会被丢弃：

    <?php

    class App\Jobs\ShipOrder;

    class ExampleTest extends TestCase
    {
        /**
         * Test order cancellation.
         */
        public function testOrderCancellation()
        {
            $this->withoutJobs();

            // Test order cancellation...
        }
    }

<a name="mocking-facades"></a>
## 假面

不像传统的静态方法的调用，[假面](/{{language}}/{{version}}/facades) 是可以被模拟的。如果你使用依赖注入的话，那么它相对于传统的静态方法来说更是具有极大的优势，同时也提供了更高的可测试性。当测试时，你或许经常需要模拟一个 Laravel 假面的调用。比如，考虑一下下面的控制器操作：

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

我们可以使用 `shouldReceive` 方法来模拟调用 `Cache` 假面，它会返回一个 [Mockery](https://github.com/padraic/mockery) 的模拟实例。由于假面实际上是通过 Laravel 的 [服务容器](/{{language}}/{{version}}/container) 来解析和管理的，所以它们比一般的静态类更具可测试性。比如，让我们来模拟 `Cache` 假面的 `get` 方法的调用：

    <?php

    class FooTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');

            $this->visit('/users')->see('value');
        }
    }

> {note} 你不应该模拟 `Request` 假面。你应该在运行测试时使用 HTTP 帮助方法如 `call` 和 `post` 来传递你所希望的输入。
