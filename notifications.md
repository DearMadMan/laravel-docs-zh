# 通知

- [前言](#introduction)
- [创建通知](#creating-notifications)
- [发送通知](#sending-notifications)
    - [使用 Notifiable Trait](#using-the-notifiable-trait)
    - [使用 Notification 假面](#using-the-notification-facade)
    - [指定交付渠道](#specifying-delivery-channels)
    - [异步队列通知](#queueing-notifications)
- [邮件通知](#mail-notifications)
    - [格式化邮件信息](#formatting-mail-messages)
    - [自定义收件人](#customizing-the-recipient)
    - [自定义主题](#customizing-the-subject)
    - [自定义模板](#customizing-the-templates)
    - [错误消息](#error-messages)
- [数据库通知](#database-notifications)
    - [依赖](#database-prerequisites)
    - [格式化数据库通知](#formatting-database-notifications)
    - [访问通知](#accessing-the-notifications)
    - [标记为已读](#marking-notifications-as-read)
- [广播通知](#broadcast-notifications)
    - [依赖](#broadcast-prerequisites)
    - [格式化广播通知](#formatting-broadcast-notifications)
    - [监听通知](#listening-for-notifications)
- [短信通知](#sms-notifications)
    - [依赖](#sms-prerequisites)
    - [格式化短信通知](#formatting-sms-notifications)
    - [自定义来源号码](#customizing-the-from-number)
    - [路由短信通知](#routing-sms-notifications)
- [Slack 通知](#slack-notifications)
    - [依赖](#slack-prerequisites)
    - [格式化 Slack 通知](#formatting-slack-notifications)
    - [路由 Slack 通知](#routing-slack-notifications)
- [通知事件](#notification-events)
- [自定义渠道](#custom-channels)

<a name="introduction"></a>
## 前言

除了支持 [发送邮件](/{{language}}/{{version}}/mail) 之外，Laravel 也支持多种交付渠道的通知行为。其中包括邮件，短信（通过 [Nexmo](https://www.nexmo.com/) ），和 [Slack](https://slack.com)。通知也可以被存储于数据库中，这样它们就可以在你的 web 接口中用于展示。

通常，通知应该是告知用户应用中发生了某些事情的短消息信息。比如，如果你在编写一个计费应用，那么你可能会需要通过邮件或者短信的渠道来为你的用户发送一个 "账单已支付" 的通知。

<a name="creating-notifications"></a>
## 创建通知

在 Laravel 中，每个通知都被一个单独的类所表示（通常被存储在 `app/Notifications` 目录下）。如果你的应用中没有这个目录，请不要担心，它会在你执行 `make:notification` Artisan 命令时生成：

    php artisan make:notification InvoicePaid

这个命令会在你的 `app/Notifications` 目录下生成一个新的通知类。所有的通知类都会包含一个 `via` 方法和各种消息的构建方法（比如 `toMail` 或者 `toDatabase`），这些方法为指定的渠道转换通知到优化的消息。

<a name="sending-notifications"></a>
## 发送通知

<a name="using-the-notifiable-trait"></a>
### 使用 Notifiable Trait

通知可以通过两个方式进行发送：使用 `Notifiable` trait 的 `notify` 方法，或者使用 `Notification` [假面](/{{language}}/{{version}}/facades)。首先，让我们来检验一下 `Notifiable` trait。这个性状被默认的 `App\User` 模型所引入，并且它提供了一个方法用于发送通知：`notify`。`notify` 方法接收一个通知类实例作为参数：

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> {tip} 请牢记，你可以在你的任意模型中使用 `Illuminate\Notifications\Notifiable` trait。你并没有被限制只在 `User` 模型中使用它。

<a name="using-the-notification-facade"></a>
### 使用 Notification 假面

另外，你也可以通过使用 `Notification` [假面](/docs/{{version}}facades) 来发送通知。这主要用于当你向多个可通知的实体发送通知时使用，比如一次性发送多个用户的通知，这个时候你需要传递所有可通知的实体和通知实例到 `send` 方法中：

    Notification::send($users, new InvoicePaid($invoice));

<a name="specifying-delivery-channels"></a>
### 指定交付渠道

每个通知类都包含 `via` 方法用于来判断使用哪个通知渠道进行通知交付行为。Laravel 本身已经对 `mail`，`database`，`broadcast`，`nexmo` 和 `slack` 渠道提供了无缝的支持。

> {tip} 如果你希望使用其他的驱动进行通知交付行为，比如 Telegram 或者 Pusher，那么你可以看看社区驱动 [Laravel Notification Channels website](http://laravel-notification-channels.com) 。

`via` 方法接收一个 `$notifiable` 实例，它应该是一个即将通知到的实体类的一个实例。你可以使用 `$notifiable` 来判断通知应该使用哪些渠道进行交付：

    /**
     * Get the notification's delivery channels.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function via($notifiable)
    {
        return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### 异步队列通知

> {note} 在你队列化通知之前，你应该先配置你的队列并且 [启动一个队列工人](/{{language}}/{{version}}/queues)

发送通知可能会消耗一些时间，特别是当你需要调用外部的 API 来交付你的通知时。为了能快速的响应，你可以让你的通知类实现 `ShouldQueue` 接口和引入 `Queueable` trait 来使其队列化。在使用 `make:notification` 所生成的通知类中已经导入了这个接口和 trait，所以你只需要直接添加它们到你的通知类就可以了：

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

当 `ShouldQueue` 接口被添加到你的通知类之后，你就可以像平常一样发送通知了。Laravel 会指定的探测类中的 `ShouldQueue` 接口，并且它会自动的队列化交付通知：

    $user->notify(new InvoicePaid($invoice));

如果你希望延迟交付通知，那么你可以在通知实例中链式的调用 `delay` 方法：

    $when = Carbon::now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($when));

<a name="mail-notifications"></a>
## 邮件通知

<a name="formatting-mail-messages"></a>
### 格式化邮件通知

如果一个通知需要支持邮件类型的交付，那么你需要在通知类中定义 `toMail` 方法。这个方法将会接收一个 `$notifiable` 实体，并且它应该返回一个 `Illuminate\Notifications\Messages\MailMessage` 实例。邮件消息可以包含数行文本及 "跳转的行为"。让我们来看一下 `toMail` 的示例：

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->line('One of your invoices has been paid!')
                    ->action('View Invoice', $url)
                    ->line('Thank you for using our application!');
    }

> {tip} 注意上面，在我们的 `message` 方法中使用了 `$this->invoice->id` 。你可以传递任意通知生成消息所需的数据到你的通知类构造函数中。

在这个例子中，我们注册了一行文本，一个跳转动作，和另外一行文本。这些由 `MailMessage` 对象所提供的方法可以方便快捷的格式化小型事务性的邮件。邮件驱动将会继续转义消息组件到一个美观的响应式 HTML 邮件模板和一个纯文本中。这里有一个 `mail` 渠道所生成邮件的示例:

<img src="https://laravel.com/assets/img/notification-example.png" width="551" height="596">

<a name="customizing-the-recipient"></a>
### 自定义收件人

当使用 `mail` 渠道发送通知时，通知系统会自动的在你的待通知实体类中查找 `email` 属性。你可以通过在你的实体中定义 `routeNotificationForMail` 方法来自定义交付通知所需要的邮件地址：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the mail channel.
         *
         * @return string
         */
        public function routeNotificationForMail()
        {
            return $this->email_address;
        }
    }

<a name="customizing-the-subject"></a>
### 自定义主题

默认的，邮件的主题只是简单的对通知类名进行了格式化。所以，如果你通知类被命名为 `InvoicePaid`，那么邮件主题将会是 `Invoice Paid`。如果你需要为你的消息指定一个明确的主题，那么你可以在你构建消息时调用 `subject` 方法:

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-template"></a>
### 自定义模板

你可以对发布的通知扩展包资源中的 HTML 和 纯文本模板进行修改已达到定制化模板的目的。在你执行完发布命令之后，邮件通知模板将会被存放在 `resources/views/vendor/notifications` 目录：

    php artisan vendor:publish --tag laravel-notifications

<a name="error-messages"></a>
### 错误消息

有些通知是为了向用户提供错误消息。比如一个失败的支付消息。你可以在构建你的消息时使用 `error` 方法来明确的指出这是一个关于错误的消息。当在一个邮件消息中使用 `error` 方法时，用于跳转动作的按钮将会使用红色来替代蓝色：

    /**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Message
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->error()
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="database-notifications"></a>
## 数据库通知

<a name="database-prerequisites"></a>
### 依赖

`database` 通知渠道会将通知的信息存储到数据库的表中。这个表将会包含通知中的一些信息，比如通知的类型以及其它用于描述通知的自定义数据的 JSON 格式。

你可以在应用中的用户接口中查询这个表来展示通知的信息。但是，在你可以做到这些之前，你需要先创建一个数据表来处理这些通知。你可以使用 `notifications:table` 命令来生成适当的迁移表结构:

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### 格式化数据库通知

如果一个通知是存储于数据库表中的，那么你应该在你的通知类中定义 `toDatabase` 或者 `toArray` 方法。这个方法将会接收一个 `$notifiable` 实体，并且它应该返回一个原生的 PHP 数组。被返回的数组将会被进行 JSON 格式化并存储到你的 `notifications` 表中的 `data` 列中。让我们来看一下 `toArray` 方法的示例：

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

#### `toDatabase` Vs. `toArray`

`toArray` 方法也被用于 `broadcast` 渠道，用于决定哪些数据应该广播到你的 JavaScript 客户端。如果你希望分别对 `database` 和 `broadcast` 渠道定义不同的交付数据，那么你应该定义 `toDatabase` 方法来取代 `toArray` 方法。

<a name="accessing-the-notifications"></a>
### 访问通知

当通知被存储到数据库之后，你需要一种方便的放法从你的可通知实体中访问它们。那么你可以使用 `Illuminate\Notifications\Notifiable` trait，它被 Laravel 的默认的 `App\User` 模型所引入，它包含了一个 `notifications` Eloquent 关联用于返回被通知实体中的所有的通知。为了获取这些通知，你可以像访问其它 Eloquent 关联模型一样的方式来访问这个方法。默认的，通知会被经过 `create_at` 排序：

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

如果你希望只检索那些 "未读" 的通知，那么你可以使用 `unreadNotifications` 关联方法。跟上面一样，这些通知会被经过 `created_at` 排序：

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} 为了能在你的 JavaScript 客户端中访问你的通知，你应该定义一个用于通知的控制器，它经用户在通知时返回一个被通知的实体，比如当前的用户。然后你就可以在你的 JavaScript 客户端构建一个到控制器 URL 的 HTTP 请求了。

<a name="marking-notifications-as-read"></a>
### 将通知设为已读

通常的，你将会在用户已经浏览过通知内容之后将其设置为已读状态。`Illumiante\Notifications\Notifiable` trait 提供了 `markAsRead` 方法，它会更新数据库中通知记录的 `read_at` 列：

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

事实上，为了避免循环每个通知，你可以在你的通知集合中直接调用 `markAsRead` 方法：

    $user->unreadNotifications->markAsRead();

你也可以使用查询更新的方式来标记所有通知为已读，并且这还不需要从数据库中检索它们:

    $user = App\User::find(1);

    $user->unreadNotifications()->update(['read_at' => Carbon::now()]);

当然，你也可以使用 `delete` 方法从数据库中删除这些通知：

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## 广播通知

<a name="broadcast-prerequisites"></a>
### 依赖

在进行广播通知之前，你应该先配置和熟悉 Laravel 的 [事件广播](/{{language}}/{{version}}/broadcasting) 服务。事件广播为你的 JavaScript Client 接收服务端所触发的 Laravel 事件提供了一种途径。

<a name="formatting-broadcast-notifications"></a>
### 格式化广播通知

`broadcast` 渠道的广播通知使用了 Laravel 的 [事件广播](/{{language}}/{{version}}/broadcasting) 服务，这允许你的 JavaScript 客户端可以实时的捕捉到通知信息。如果通知支持广播的形式，那么你需要在通知类中定义 `toBroadcast` 或者 `toArray` 方法。这些方法将会接收一个 `$notifiable` 实体并且返回一个原生的 PHP 数组。被返回的数组将会经过 JSON 格式化并被广播到你的 JavaScript 客户端。让我们来看一下 `toArray` 方法的示例：

    /**
     * Get the array representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

> {tip} 除了你所指定的数据之外，广播通知也会包含一个 `type` 字段用以包含通知类的名字。

#### `toBroadcast` Vs. `toArray`

`toArray` 方法也被用于 `database` 渠道，用来判断哪些数据应该存储于数据表中。如果你希望分别为 `database` 和 `broadcast` 渠道分配不同的交付数据，那么你应该定义 `toBroadcast` 方法来取代 `toArray` 方法。

<a name="listening-for-notifications"></a>
### 监听通知

通知将会以 `{notifiable}.{id}` 所约定的格式经过一个私有频道进行广播。所以，如果你发送一个通知到 `App\User` 模型实例 ID 为 1 的用户，那么通知将会被通过 `App.User.1` 私有频道进行广播。当使用 [Laravel Echo](/{{language}}/{{version}}/broadcasting) 时，你可以在频道中使用 `notification` 帮助方法来轻松的监听通知：

    Echo.private('App.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

<a name="sms-notifications"></a>
## 短信通知

<a name="sms-prerequisites"></a>
### 依赖

Laravel 的短信通知时由 [Nexmo](https://www.nexmo.com/) 所提供的。在你可以通过 Nexmo 发送通知之前，你应该先通过 Composer 安装 `nexmo/client` 包，并且你需要在你的 `config/services.php` 配置文件中添加一些额外的配置选项。你可以直接复制下面的配置示例：

    'nexmo' => [
        'key' => env('NEXMO_KEY'),
        'secret' => env('NEXMO_SECRET'),
        'sms_from' => '15556666666',
    ],

`sms_from` 选项用于指定短信消息的来源号码。你应该在 Nexmo 控制面板中生成这个电话号码。

<a name="formatting-sms-notifications"></a>
### 格式化短信通知

如果通知支持通过短信渠道发送，那么你应该在你的通知类中定义 `toNexmo` 方法。这个方法将会接收 `$notifiable` 实体并且它应该返回一个 `Illuminate\Notifications\Message\NexmoMessage` 实例：

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content');
    }

<a name="customizing-the-from-number"></a>
### 自定义来源号码

如果你希望在发送一些通知时使用与 `config/services.php` 配置文件中所指定的电话号码不同的号码，那么你可以在 `NexmoMessage` 实例中使用 `from` 方法：

    /**
     * Get the Nexmo / SMS representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>
### 路由短信通知

当使用 `nexmo` 渠道发送通知时，通知系统会自动的在被通知实体中查找 `phone_number` 属性。如果你希望自定义交付的电话号码，那么你可以在实体类中定义 `routeNotificationForNexmo` 方法：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Nexmo channel.
         *
         * @return string
         */
        public function routeNotificationForNexmo()
        {
            return $this->phone;
        }
    }

<a name="slack-notifications"></a>
## Slack 通知

<a name="slack-prerequisites"></a>
### 依赖

在你可以通过使用 Slack 发送通知以前，你需要先通过 Composer 安装 Guzzle HTTP 类库：

    composer require guzzlehttp/guzzle

你将会需要为你的 Slack 团队配置一个 "Incoming Webhook"。这种整合将会在你进行 [路由 Slack 通知](#routing-slack-notifications) 提供一个可用的 URL。

<a name="formatting-slack-notifications"></a>
### 格式化 Slack 通知

如果一个通知支持 Slack 消息渠道，那么你需要在通知类中定义 `toSlack` 方法。这个方法将会接收一个 `$notifiable` 实体，并且它应该返回一个 `Illuminate\Notifications\Messages\SlackMessage` 实例。Slack 消息可以包含文本内容以及一个格式化文本或数组字段附件。让我们来看一下 `toSlack` 示例：

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('One of your invoices has been paid!');
    }

在这个例子中我们只是发送了一行文本到 Slack，它将会创造像下面的消息：

<img src="https://laravel.com/assets/img/basic-slack-notification.png">

#### Slack 附件

你也可以添加 "附件" 到 Slack 消息中。附件可以提供比简单的文本消息更丰富的格式化选项。在这个例子中，我们将会发送一个关于应用中所发生的异常的信息的通知，其中还包含了一个导向异常详情的连接：

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('Whoops! Something went wrong.')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Exception: File Not Found', $url)
                                   ->content('File [background.jpg] was not found.');
                    });
    }

上面的示例所生成的 Slack 消息看起来像这样：

<img src="https://laravel.com/assets/img/basic-slack-attachment.png">

附件也允许你指定一个数组数据来呈现给用户。所给定的数据将会经过表格风格的格式化来呈现，这样更易于阅读：

    /**
     * Get the Slack representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/invoices/'.$this->invoice->id);

        return (new SlackMessage)
                    ->success()
                    ->content('One of your invoices has been paid!')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('Invoice 1322', $url)
                                   ->fields([
                                        'Title' => 'Server Expenses',
                                        'Amount' => '$1,234',
                                        'Via' => 'American Express',
                                        'Was Overdue' => ':-1:',
                                    ]);
                    });
    }

上面示例所创建的 Slack 消息看起来像下面一样：

<img src="https://laravel.com/assets/img/slack-fields-attachment.png">

<a name="routing-slack-notifications"></a>
### 路由 Slack 通知

为了路由 Slack 通知到适当的位置，你可以在你的被通知实体类中定义 `routeNotificationForSlack` 方法。这个方法应该返回一个 webhook URL，它代表了通知应该交付的地址。Webhook URLs 可以在你的 Slack 团队中的 `Incoming Webhook` 服务中生成：

    <?php

    namespace App;

    use Illuminate\Notifications\Notifiable;
    use Illuminate\Foundation\Auth\User as Authenticatable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Route notifications for the Slack channel.
         *
         * @return string
         */
        public function routeNotificationForSlack()
        {
            return $this->slack_webhook_url;
        }
    }

<a name="notification-events"></a>
## 通知事件

当通知被发送时，通知系统将会触发 `Illuminate\Notifications\Events\NotificationSent` 事件。这个事件包含了 "notifiable" 实体和通知自身的实例。你可以在你的 `EventServiceProvider` 中注册这个事件的监听者：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} 在你的 `EventServiceProvider` 中注册完成监听者之后，你可以使用 `event:generate` Artisan 命令来快速的生成监听者类。

在事件监听者中，你可以访问事件实例中的 `notifiable`，`notification` 和 `channel` 属性来了解更多关于通知接收者或者通知本身的信息：

    /**
     * Handle the event.
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="custom-channels"></a>
## 自定义渠道

Laravel 只是附带了少数的几种通知渠道，但是你可以编写自己的驱动来通过其它渠道交付你的通知。Laravel 可以非常简单的做到这些。在开始之前，你需要定义一个包含 `send` 方法的类。这个方法应该接收两个参数：一个 `$notifiable` 和一个 `$notification`：

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * Send the given notification.
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // Send notification to the $notifiable instance...
        }
    }

当你的通知渠道类被定义完成之后，你就可以简单的在你的通知类中的 `via` 方法中返回这个类的名称以使用这个渠道:

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use App\Channels\VoiceChannel;
    use App\Channels\Messages\VoiceMessage;
    use Illuminate\Notifications\Notification;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * Get the notification channels.
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

        /**
         * Get the voice representation of the notification.
         *
         * @param  mixed  $notifiable
         * @return VoiceMessage
         */
        public function toVoice($notifiable)
        {
            // ...
        }
    }
