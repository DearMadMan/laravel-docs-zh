# 事件广播

- [前言](#introduction)
    - [配置](#configuration)
    - [驱动需求](#driver-prerequisites)
- [概念概述](#concept-overview)
    - [应用示例](#using-example-application)
- [定义广播事件](#defining-broadcast-events)
    - [广播数据](#broadcast-data)
    - [广播队列](#broadcast-queue)
- [授权频道](#authorizing-channels)
    - [定义授权路由](#defining-authorization-routes)
    - [定义授权回调](#defining-authorization-callbacks)
- [广播事件](#broadcasting-events)
    - [排除当前用户](#only-to-others)
- [接收广播](#receiving-broadcasts)
    - [安装 Laravel Echo](#installing-laravel-echo)
    - [监听事件](#listening-for-events)
    - [命名空间](#namespaces)
- [会场频道](#presence-channels)
    - [授权会场频道](#authorizing-presence-channels)
    - [加入会场频道](#joining-presence-channels)
    - [广播到会场频道](#broadcasting-to-presence-channels)
- [通知](#notifications)

<a name="introduction"></a>
## 前言

很多现代化的应用中，会使用 WebSockets 来实现实时交互的用户接口。当一些数据在服务端变更时，一条消息会通过 websocket 连接来传递到客户端进行处理。这相比较轮询的方式来说更为健壮有效。

为了帮助你构建这种类型的应用。Laravel 使通过 WebSocket 连接进行广播事件变的非常简单。Laravel 允许你通过 WebSocket 广播 [事件](/docs/{{language}}/{{version}}/events) 来共享事件的名称到你的服务端和客户端的 JavaScript 框架。

> {tip} 在我们更深入的探讨事件广播之前，请确认你已经读完了关于 Laravel 的 [事件和监听器](/docs/{{language}}/{{version}}/events) 文档。

<a name="configuration"></a>
### 配置

所有的事件广播配置选项都被存储在 `config/broadcasting.php` 配置文件中。Laravel 支持多种开箱即用的广播驱动：[Pusher](https://pusher.com/)，[Redis](/docs/{{language}}/{{version}}/redis) 和 `log` 驱动来提供本地开发和调试。另外，还有一个 `null` 驱动用于完全的禁用广播。在这个配置文件中包含了每种驱动的配置示例。

#### 广播服务提供者

在你广播任意事件之前，你首先需要注册一个 `App\Providers\BroadcastServiceProvider`。在一个新的 Laravel 应用中，你只需要在 `config/app.php` 配置文件中的 `providers` 数组中取消这个提供者的注释就可以了。这个提供者将会允许你注册广播的授权路由和回调函数。

#### CSRF Token

[Laravel Echo](#installing-laravel-echo) 将会访问当前会话的 CSRF token。如果是可用的，那么 Echo 会从 `Laravel.csrfToken` JavaScript 对象中获取到它。这个对象被定义在了 `resources/views/layouts/app.blade.php` 布局文件中，它只会在你执行 `make:auth` Artisan 命令之后才会被创建。如果你不使用这个布局，那么你需要在应用的 `head` 元素内定义一个 `meta` 标签来包含 token:

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### 驱动需求

#### Pusher

如果你使用 [Pusher](https://pusher.com) 来广播你的事件，那么你应使用 Composer 安装 Pusher PHP SDK:

    composer require pusher/pusher-php-server

然后，你应该在 `config/broadcasting.php` 配置文件中配置你的 Pusher 相关的凭证。在这个配置文件中一个 Pusher 配置的示例已经被引入了，你可以快速的指定你的 Pusher Key，密钥，和应用 ID。

当同时使用 Pusher 和 [Laravel Echo](#installing-laravel-echo) 时，你应该在实例化 Echo 实例时指定 `pusher` 为你所期望使用的广播器:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

#### Redis

如果你使用的是 Redis 广播器，那么你应该先安装 Predis 类库：

    composer require predis/predis

Redis 将利用 Redis 自身的发布 / 订阅机制来广播信息。然而，你却需要相应的 WebSocket 服务端来接收 Redis 所发布的消息，并且将其广播到你的 WebSocket 频道中。

当 Redis 广播发布一个事件时，它将会在事件所指定的频道中去发布，并且载荷 `data` 的数据是一个被 JSON 化的字符串，其中就包括了事件的名称和用户。

#### Socket.IO

如果你选择使用 Socket.IO 来配合 Redis 广播，那么你需要在应用的 `head` 元素中引入 Socket.IO 的 JavaScript 客户端类库:

    <script src="https://cdn.socket.io/socket.io-1.4.5.js"></script>

然后，你将会需要使用 `socket.io` 连接器和 `host` 来实例化 Echo。比如，如果你的应用和 socket 服务运行在 `app.dev` 域名下，那么你应该像下面这样实例化 Echo:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: 'http://app.dev:6001'
    });

最后，你需要运行一个具有兼容性的 Socket.IO 服务端。Laravel 并没有引入 Socket.IO 服务端的实现。但是，社区产出的一个 Socket.IO 服务端驱动当前被维护在 [tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) Github 仓库中。

#### 队列依赖

在进行事件广播之前，你需要先配置好 [队列监听器](/docs/{{language}}/{{version}}/queues)。所有的广播都是通过队列任务来异步执行的，这样应用响应就不会受到影响。

<a name="concept-overview"></a>
## 概念概述

Laravel 的事件广播允许你通过基于驱动的途径在 WebSockets 通道中将服务端的 Laravel 事件广播到你的客户端的 JavaScript 应用中。目前，Laravel 自带了 [Pusher](http://pusher.com) 和 Redis 驱动。你可以使用 [Laravel Echo](#installing-laravel-echo) JavaScript 包来轻松的对接这些事件。

事件通过频道进行广播。频道其实是可以被指定为私有或公开的。应用中的任意访客都可以在未授权或者认证的情况下订阅公开的频道。但是，为了订阅一些私人的频道时，用户就必须是已经认证并且被授权监听于此频道才行。

<a name="using-example-application"></a>
### 应用示例

在深入了解事件广播的每个组件之前，先让我们来看一个更高级的关于电子商务平台的一个示例。我们不会探讨配置 [Pusher](http://pusher.com) 或者 [Laravel Echo](#echo) 的细节，因为这些我们会在文档中的其它区域中详细的进行了解。

在我们的应用中，让我们先假设我们有一个页面是允许用户查看他们订单的配送状态的。那么我们也假设当应用处理更新配送状态时，我们会触发一个 `ShippingStatusUpdated` 事件:

    event(new ShippingStatusUpdated($update));

#### `ShouldBroadcast` 接口

当用户在浏览他们的订单时，我们不想他们重新刷新页面才能看到新的订单状态。那么，我们需要广播应用所更新的状态。我们需要让 `ShippingStatusUpdated` 事件实现 `ShouldBroadcast` 接口，这将会指导 Laravel 当事件触发时将其广播出去：

    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ShippingStatusUpdated implements ShouldBroadcast
    {
        //
    }

`ShouldBroadcast` 接口要求我们的事件必须定义了 `broadcastOn` 方法。这个方法的职责就是返回事件应该被广播到哪些频道中。在使用命令生成的事件类中，Laravel 已经为这个方法填充了一些桩代码。所我们只需要填充一些详情就可以了。我们只需要订单所属的用户来浏览状态的更新，所以我们将事件通过一个私有的频道来绑定到订单，然后将其广播出去:

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array
     */
    public function broadcastOn()
    {
        return new PrivateChannel('order.'.$this->update->order_id);
    }

#### 授权频道

还记得吗，用户必须被授权才能监听私有的频道。我们可以在 `BroadcastServiceProvider` 的 `boot` 方法中定义我们的频道授权约束。在这个例子中，我们需要验证任意尝试去监听私有的 `order.1` 频道的用户是不是这个订单的所属用户:

    Broadcast::channel('order.*', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel` 方法接收两个参数：频道的名称，和一个判断回调，这个回调需要明确的返回 `true` 或者 `false`，用于表示是否被授权监听此频道。

所有的授权回调都会接收当前已认证的用户作为它们的第一个参数，并且任意额外的通配符参数将作为它们的后续参数。在这个例子中，我们使用 `*` 字符来表示频道中的 "ID" 部分通配符匹配。

#### 监听事件广播

接着，剩下的就是在我们的 JavaScript 应用中监听事件了。我们可以使用 Laravel Echo 做这些。首先，我们将使用 `private` 方法来订阅私有频道。然后，我们可以使用 `listen` 方法来监听 `ShippingStatusUpdate` 事件。默认的，所有的事件中公开属性都会被在广播事件时作为载荷传递：

    Echo.private('order.' + orderId)
        .listen('ShippingStatusUpdated', (e) => {
            console.log(e.update);
        });

<a name="defining-broadcast-events"></a>
## 定义广播事件

你需要在事件类中实现 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 接口，以通知 Laravel 所给定的事件需要被广播。这个接口在生成所有事件类时都已经被框架引入了，所以你可以轻松的添加它到你的事件类中。

`ShouldBroadcast` 接口只要求你实现一个单一的方法：`broadcastOn`。该方法应该返回一个事件应该被广播的频道实例或者是频道实例所组成的数组。频道应该是 `Channel`，`PrivateChannel` 或者 `PresenceChannel` 中的一个实例。对于 `Channel` 所代表的公开频道，任意用户都可以订阅，而对于 `PrivateChannels` 和 `PresenceChannels` 所代表的私人频道，只有被 [授权的频道](#authorizing-channels) 才可以被订阅：


    <?php

    namespace App\Events;

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

    class ServerCreated implements ShouldBroadcast
    {
        use SerializesModels;

        public $user;

        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }

        /**
         * Get the channels the event should broadcast on.
         *
         * @return Channel|array
         */
        public function broadcastOn()
        {
            return new PrivateChannel('user.'.$this->user->id);
        }
    }

任何，你只需要像平常一样 [触发事件](/docs/{{language}}/{{version}}/events) 就可以了。当事件被触发时，一个 [队列任务](/docs/{{language}}/{{version}}/queues) 会自动的依据你指定的广播驱动来广播事件。

<a name="broadcast-data"></a>
### 广播数据

当事件被广播时，事件中所有的 `public` 属性都会被自动的序列化，然后序列化后的内容将会作为事件的载荷被广播出去。这允许你在前端的 JavaScript 应用中访问这些公开的数据。那么，我们来举个例子，如果你的事件中只有一个 `public` 属性 `$user`，它是一个 Eloquent 模型，那么事件广播的载荷将会像这样：

    {
        "user": {
            "id": 1,
            "name": "Patrick Stewart"
            ...
        }
    }

但是，如果你希望对这些载荷有更多的控制权，你可以在你的事件类中添加 `broadcastWith` 方法。这个方法应该返回一个数组，数组将会作为事件的载荷被广播出去:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['id' => $this->user->id];
    }

<a name="broadcast-queue"></a>
### 广播队列

默认的，所有的广播事件都是依据 `queue.php` 配置文件中的队列连接被推送到默认队列中的。你可以在事件类中定义 `broadcastQueue` 属性来指定广播应该使用哪个队列。这个属性应该指定当该事件被广播时所使用的队列的名称:

    /**
     * The name of the queue on which to place the event.
     *
     * @var string
     */
    public $broadcastQueue = 'your-queue-name';

<a name="authorizing-channels"></a>
## 授权频道

私有频道要求你对当前认证的用户授权是否可以监听该频道。当 HTTP 请求流入 Laravel 应用时会伴随频道的名称，这时 Laravel 会依据私有频道的性质来判断当前用户是否有监听此频道的权限。当使用 [Laravel Echo](#installing-laravel-echo) 时，HTTP 请求订阅私有频道会自动的完成。不然的话，你需要构建适当的路由来对这些请求作出响应。

<a name="defining-authorization-routes"></a>
### 定义授权路由

非常帮！Laravel 可以轻松的为这些频道授权请求定义响应路由。在你的 Laravel 应用的 `BroadcastServiceProvider` 中，你可以看到 `Broadcast::routes` 方法。这个方法将会注册 `/broadcasting/auth` 路由来处理授权请求：

    Broadcast::routes();

`Broadcast::routes` 方法会自动的将它的路由放置在 `web` 中间件组中。然而，你可以传递路由的属性所组成的数组到这个方法中来分配定制化的属性:

    Broadcast::routes($attributes);

<a name="defining-authorization-callbacks"></a>
### 定义授权回调

接着，我们需要定义将提供给频道授权的逻辑。就像定义授权路由那样，这些都可以在 `BroadcastServiceProvider` 的 `boot` 方法中完成。在这个方法中，你可以使用 `Broadcast::channel` 方法来注册频道授权回调：

    Broadcast::channel('order.*', function ($user, $orderId) {
        return $user->id === Order::findOrNew($orderId)->user_id;
    });

`channel` 方法接收两个参数：频道的名称和回调函数，回调应该明确的返回 `true` 或者 `false`，用来指明用户是否被授权监听该频道。

所有的授权回调都会接收当前已认证的用户作为它们的第一个参数，并且任意额外的通配符参数将作为它们的后续参数。在这个例子中，我们使用 `*` 字符来表示频道中的 "ID" 部分通配符匹配。

<a name="broadcasting-events"></a>
## 广播事件

当你定义了一个事件类，并且使它实现了 `ShouldBroadcast` 接口，那么你只需要使用 `event` 方法来触发这个事件，事件分发器就会通知应用这个事件已经被标记了 `ShouldBroadcast` 接口，并且应该将这个事件放到广播队列中去：

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### 排除当前用户

当利用事件广播来构建一个应用时，你可以使用 `broadcast` 方法来替代 `event`。就像 `event` 方法，`broadcast` 方法会分发事件到你的服务端监听器中:

    broadcast(new ShippingStatusUpdated($update));

但是，`broadcast` 方法也公开了 `toOthers` 方法，这允许你从广播的接受者中排除当前用户:

    broadcast(new ShippingStatusUpdated($update))->toOthers();

为了让你更好的理解我们应该什么时候使用 `toOthers` 方法，那么让我们来想象一下有这么一个任务列表应用。当用户输入一个任务名称之后，他可以创建一个新的任务。为了创建一个任务，你的应用应该发送了一个请求到 `/task`，它应该广播任务的创建信息并且返回一个 JSON 来代表新的任务。当你的 JavaScript 应用从服务端接收这个响应时，它应该像下面这样直接的在任务列表中加入了新的任务:

    this.$http.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

但是，还记得我们也广播了任务的创建信息吗。如果你的 JavaScript 应用为了在任务列表中添加新的任务而监听这个事件，那么你将会拥有一个重复的任务：一个来自广播事件，一个来自服务端的响应。

你可以使用 `toOthers` 方法来解决这个冲突，它将会指导广播器不要对当前的用户广播这个事件。

#### 配置

当你实例化 Laravel Echo 实例时，一个 socket ID 会被分配到连接中。如果你使用 [Vue](https://vuejs.org) 和 Vue Resource，那么 socket ID 会在每一个即将离开的请求中附加到 `X-Socket-ID` 请求头。然后你可以调用 `toOthers` 方法，Laravel 会自动的从请求头中提取 socket ID，然后指导广播器不要向这个 socket ID 的连接中广播事件。

如果你没有使用 Vue 和 Vue Resource，那么你将需要手动的配置你的 JavaScript 应用来发送 `X-Socket-ID` 请求头。你可以使用 `Echo.socketId` 来获取 socket ID：

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## 接收广播

<a name="installing-laravel-echo"></a>
### 安装 Laravel Echo

Laravel Echo 是一个 JavaScript 类库。它可以无缝的订阅频道和监听 Laravel 所广播的事件。你可以通过 NPM 包管理器来安装 Echo。在这个示例中，我们也会安装 `pusher-js` 包，因为我们将会使用 Pusher 广播器:

    npm install --save laravel-echo pusher-js

当 Echo 安装完成之后，你可以在你应用的 JavaScript 中创建一个新的 Echo 实例。我们推荐你在 `resources/assets/js/bootstrap.js` 文件中来做这些:

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key'
    });

<a name="listening-for-events"></a>
### 监听事件

当你安装完成 Echo 并实例化之后，你可以开始监听事件广播了。首先，你可以使用 `channel` 方法来获取一个频道的实例，然后调用 `listen` 方法来监听指定的事件：

    Echo.channel('orders')
        .listen('OrderShipped', (e) => {
            console.log(e.order.name);
        });

如果你需要监听一个私有频道的事件，那么你需要使用 `private` 方法。你可以持续的链式调用 `listen` 方法来在一个频道中监听多个事件：

    Echo.private('orders')
        .listen(...)
        .listen(...)
        .listen(...);

<a name="namespaces"></a>
### 命名空间

你可能已经注意到了我们上面的示例中并没有指定事件类的完整的命名空间。这是因为 Echo 将会自动的假定事件类被存放在 `App\Events` 命名空间下。但是，你可以在实例化 Echo 时传递 `namespace` 配置选项来设置根命名空间:

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-key',
        namespace: 'App.Other.Namespace'
    });

另外，当你使用 Echo 订阅这些事件时，你也可以使用 `.` 作为事件类的前缀，这允许你总是指定完整的类名:

    Echo.channel('orders')
        .listen('.Namespace.Event.Class', (e) => {
            //
        });

<a name="presence-channels"></a>
## 会场频道

会场频道基于安全的私有频道，但是它却会为其它的订阅该频道的用户暴露出一些额外的特性。这种特性可以非常轻松的构建出强大的协作应用。比如，当一个用户浏览一个页面时，对其他浏览该页面的用户做出通知。

<a name="joining-a-presence-channel"></a>
### 授权会场频道

所有的会场频道都是私有频道。因此，用户必须 [被授权才可以监听](#authorizing-channels)。但是，当为会场频道定义一个授权回调时，如果一个用户被授权加入该频道，你不应该返回 `true`，相应的你应该返回关于这个用户的相关数组。

这个授权回调所返回的数据，将会在你的 JavaScript 应用中被会场频道中的事件监听器所使用。如果用户并没有被授权加入这个会场频道，那么你应该返回 `false` 或者 `null`：

    Broadcast::channel('chat.*', function ($user, $roomId) {
        if ($user->canJoinRoom($roomId)) {
            return ['id' => $user->id, 'name' => $user->name];
        }
    });

<a name="joining-presence-channels"></a>
### 加入会场频道

为了加入一个会场频道，你应该使用 Echo 的 `join` 方法。`join` 方法将会返回一个 `PresenceChannel` 的实现，沿着被公开的 `listen` 方法，你可以订阅 `here`，`joining` 和 `leaving` 事件。

    Echo.join('chat.' + roomId)
        .here((users) => {
            //
        })
        .joining((user) => {
            console.log(user.name);
        })
        .leaving((user) => {
            console.log(user.name);
        });

`here` 回调将会在加入频道成功之后立即执行，并且它会接收一个包含用户信息的数组并将其提供与其它订阅当前频道的用户。当一个新用户加入到该频道中时，`joining` 方法将会被执行，而 `leaving` 方法会在一个用户离开频道是被执行。

<a name="broadcasting-to-presence-channels"></a>
### 广播到会场频道

会场频道可以像公开或者私有频道那样接收事件。我们来看一个关于聊天室的示例。我们可能想要广播 `NewMessage` 事件到会场频道的房间。要做到这个，我们需要从事件类的 `broadcastOn` 方法中返回一个 `PresenceChannel` 的实例：

    /**
     * Get the channels the event should broadcast on.
     *
     * @return Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->message->room_id);
    }

就像公开的或者私有的事件那样，会场频道的事件也可以通过 `broadcast` 方法来广播。正如其它的事件，你也可以使用 `toOthers` 方法来排除当前用户收到广播:

    broadcast(new NewMessage($message));

    broadcast(new NewMessage($message))->toOthers();

你可以通过使用 Echo 的 `listen` 方法来监听接入的事件：

    Echo.join('chat.' + roomId)
        .here(...)
        .joining(...)
        .leaving(...)
        .listen('NewMessage', (e) => {
            //
        });

<a name="notifications"></a>
## 通知

[通知](/docs/{{language}}/{{version}}/notifications) 总是伴随着事件广播，你的 JavaScript 应用可以在其发生时接受一个新的通知而不用刷新页面。首先，你需要确定你已经读完了使用 [广播通知频道](/docs/{{language}}/{{version}}/notifications#broadcast-notifications) 的文档。

当你为广播频道配置好通知之后，你可以使用 Echo 的 `notification` 方法来监听广播的事件。你要记得，频道名称应该与所接收到的通知的类名相匹配：

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

在这个例子中，所有通过 `broadcast` 频道发送到 `App\User` 实例的通知都将会被回调所接收到。Laravel 框架自带的 `BroadcastServiceProvider` 中已经为 `App.User.*` 频道定义了一个频道授权回调。
