# 事件

- [前言](#introduction)
- [注册事件 & 监听者](#registering-events-and-listeners)
    - [生成事件 & 监听者](#generating-events-and-listeners)
    - [手动注册事件](#manually-registering-events)
- [定义事件](#defining-events)
- [定义监听者](#defining-listeners)
- [队列化监听者](#queued-event-listeners)
    - [手动访问队列](#manually-accessing-the-queue)
- [事件触发](#firing-events)
- [事件订阅](#event-subscribers)
    - [编写事件订阅者](#writing-event-subscribers)
    - [注册事件订阅者](#registering-event-subscribers)

<a name="introduction"></a>
## 前言

Laravel 的事件提供了一种简单的观察者实现。它允许你在应用中进行订阅和监听多种事件。事件类通常都是存储在 `app/Events` 目录中，而他们的监听者都是存储在 `app/Listeners` 目录中。如果你发现自己的应用中不存在这些目录，请不要担心，它们会在你使用 Artisan 命令生成事件和监听者时自动的创建。

事件能对应用中多个方面进行有效的解耦，这是因为对于单个事件来说，它可以拥有多个互相不依赖的监听者。比如，你可能希望每次订单发货时能够发送一个 Slack 通知到你的用户，除了可以在订单处理代码中融入 Slack 发送通知代码之外，你可以非常简单的提取出一个 `OrderShipped` 事件，而它的监听者可以接收订单的变化，并将其转换成一个相应的 Slack 通知。

<a name="registering-events-and-listeners"></a>
## 注册事件 & 监听者

`EventServiceProvider` 提供了一个注册所有事件监听者的方便的场所。它的 `listen` 属性包含了一个所有事件（keys）以及他们的监听者（values）所组成的数组。当然，你可以在这个数组中添加任何你需要的事件。比如，让我们添加 `OrderShipped` 事件：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### 生成事件 & 监听者

当然，每次都手动的去创建一个事件文件和一个监听者文件是很麻烦的事情，所以，你可以在 `EventServiceProvider` 中添加事件和监听者然后使用 `event:generate` 命令来自动生成。这个命令会生成 `EventServiceProvider` 中的所有列出的事件和监听者，当然，已经存在的事件和监听者不会重新生成：

    php artisan event:generate

<a name="defining-events"></a>
## 定义事件

一个事件类只是简单的数据存储器，它应该持有事件相关的信息.比如，让我们假设我们生成的 `OrderShipped` 事件应该接收一个 [Eloquent ORM](/docs/{{language}}/{{version}}/eloquent) 对象：

    <?php

    namespace App\Events;

    use App\Order;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Event
    {
        use SerializesModels;

        public $order;

        /**
         * Create a new event instance.
         *
         * @param  Order  $order
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }
    }

就如你所看到的，这个事件类并没有包含什么业务逻辑。它只是简单的包含了一个被购买的 `Order` 对象。`SerializesModels` trait 被用来序列化 Eloquent 模型，如果事件对象是使用 PHP 的 `serialize` 方法被进行序列化，那么 `SerializesModels` 就会优雅的将其内部的 Eloquent 对象序列化。

<a name="defining-listeners"></a>
## 定义监听者

接着，让我们来看一下针对我们上面举例的事件的监听者。事件监听者会在其 `handle` 方法中接收事件的实例。`event:generate` 命令会自动的在 `handle` 方法中引入其对应的事件的类型提示。你可以在 `handle` 方法中来提供对事件的响应逻辑：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }

        /**
         * Handle the event.
         *
         * @param  OrderShipped  $event
         * @return void
         */
        public function handle(OrderShipped $event)
        {
            // Access the order using $event->order...
        }
    }

> {tip} 你的事件监听者也可以在构造函数中进行类型提示来注入依赖。所有的事件监听者都是通过 Laravel 的 [服务容器](/docs/{{language}}/{{version}}/container) 解析的。所以，它们的依赖可以被自动的注入：

#### 停止事件传递

有时候，你可能希望停止传递事件到后续的监听者中。你可以在监听者的 `handle` 方法中返回 `false`，这样后续的监听者将不再进行事件的响应。

<a name="queued-event-listeners"></a>
## 队列化监听者

队列化监听者对于那些执行比较慢的任务是十分有益的，比如发送一个邮件或者是构建一个 HTTP 请求。在开始了解队列化监听者之前，你需要确保你已经 [配置好你的队列](/docs/{{language}}/{{version}}/queues) 并且在你的服务端或者本地开发环境中已经开启了一个队列监听者。

如果需要一个队列化的监听者，那么你直接添加 `ShouldeQueue` 接口到监听者类中就可以了。通过 `event:generate` Artisan 命令生成的监听者已经在当前的命名空间中添加了对这个接口的支持，所以你立即就可以使用：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        //
    }

就这么简单！当事件触发时，事件分发器会使用 Laravel 的 [队列系统](/docs/{{language}}/{{version}}/queues) 对监听者进行自动的队列化调用。如果监听者队列化执行的过程中没有异常出现，队列任务会自动的在监听者进程完成后进行删除。

<a name="manually-accessing-the-queue"></a>
### 手动访问队列

如果你需要手动的访问底层队列任务的 `delete` 和 `release` 方法。那么你可以使用 `Illuminate\Queue\InteractsWithQueue` trait。`Illuminate\Queue\InteractsWithQueue` trait 对默认生成的监听者提供了访问这些方法的权限：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        public function handle(OrderShipped $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="firing-events"></a>
## 触发事件

如果需要触发一个事件，那么你可以传递一个事件实例到 `event` 帮助方法中。帮助方法会分发事件到所有已注册的相关监听者中。因为 `event` 帮助方法是可以全局访问的，所有你可以在应用的任意位置调用它：

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  int  $orderId
         * @return Response
         */
        public function ship($orderId)
        {
            $order = Order::findOrFail($orderId);

            // Order shipment logic...

            event(new OrderShipped($order));
        }
    }

> {tip} 当进行测试时，这对于断言那些只需要判断事件的触发而不需要相应监听者进行真正的处理是非常有帮助的。Laravel 內建的 [测试帮助方法](/docs/{{language}}/{{version}}/mocking#mocking-events) 可以很好的做到这点。

<a name="event-subscribers"></a>
## 事件订阅者

<a name="writing-event-subscribers"></a>
### 编写事件订阅者

事件订阅者允许一个类在其内部订阅多个事件。这允许你在同一个文件中对多个事件进行处理。订阅者应该定义 `subscribe` 方法。该方法会被传递一个事件分发器实例，你可以在给定的分发器中调用 `listen` 方法来注册事件监听者：

    <?php

    namespace App\Listeners;

    class UserEventSubscriber
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}

        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}

        /**
         * Register the listeners for the subscriber.
         *
         * @param  Illuminate\Events\Dispatcher  $events
         */
        public function subscribe($events)
        {
            $events->listen(
                'Illuminate\Auth\Events\Login',
                'App\Listeners\UserEventSubscriber@onUserLogin'
            );

            $events->listen(
                'Illuminate\Auth\Events\Logout',
                'App\Listeners\UserEventSubscriber@onUserLogout'
            );
        }

    }

<a name="registering-event-subscribers"></a>
### 注册事件订阅者

当你的订阅者编写完成之后，它应该在事件分发器中进行注册。你可以使用 `EventServieProvider` 的 `$subscribe` 属性来注册订阅者。比如，让我们来添加一个 `UserEventSubscriber` 到列表中:

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];

        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventSubscriber',
        ];
    }
