# 邮件

- [前言](#introduction)
    - [驱动需求](#driver-prerequisites)
- [生成 Mailables](#generating-mailables)
- [编写 Mailables](#writing-mailables)
    - [配置发送者](#configuring-the-sender)
    - [配置视图](#configuring-the-view)
    - [视图数据](#view-data)
    - [附件](#attachments)
    - [行内附件](#inline-attachments)
- [发送邮件](#sending-mail)
    - [队列化邮件](#queueing-mail)
- [邮件 & 本地环境](#mail-and-local-development)
- [事件](#events)

<a name="introduction"></a>
## 前言

Laravel 基于流行的 [SwiftMailer](http://swiftmailer.org/) 类库构建了一种干净简洁的邮件 API。Laravel 提供了驱动以支持 SMTP，Mailgun，SparkPost，Amazon SES，PHP 的 `mail` 方法和 `sendmail` 方法，这允许你快速的开始构建通过本地或云服务发送邮件的服务。

<a name="driver-prerequisites"></a>
### 驱动需求

基于 API 的驱动程序如 Mailgun 和 SparkPost 通常要比 SMTP 服务简单快速。如果可能的话，你应该尽量使用这些驱动。所有的 API 驱动都必须引入 Guzzle HTTP 类库，你可以通过在 `composer.json` 文件中添加下面的内容来进行安装：

    composer require guzzlehttp/guzzle

#### Mailgun 驱动

如果使用 Mailgun 驱动，你首先需要安装 Guzzle，然后设置 `config/mail.php` 配置文件的 `driver` 选项为 `mailgun`。然后，你需要确认你的 `config/services.php` 配置文件中包含以下选项：

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

#### SparkPost 驱动

如果使用 SparkPost 驱动，你首先需要安装 Guzzle，然后设置你的 `config/mail.php` 配置文件中的 `driver` 选项为 `sparkpost`。接着，你需要确认你的 `config/services.php` 配置文件中包含以下选项：

    'sparkpost' => [
        'secret' => 'your-sparkpost-key',
    ],

#### SES Driver

如果使用 Amazon SES 驱动，你需要在 PHP 中引入 Amazon AWS SDK。你可以通过在 `composer.json` 文件中进行引入安装：

    "aws/aws-sdk-php": "~3.0"

然后设置你的 `config/mail.php` 配置文件的 `driver` 选项为 `ses`。接着，你需要确认你的 `config/services.php` 配置文件中包含以下选项：

    'ses' => [
        'key' => 'your-ses-key',
        'secret' => 'your-ses-secret',
        'region' => 'ses-region',  // e.g. us-east-1
    ],

<a name="generating-mailables"></a>
## 生成可邮寄的类

在 Laravel 中，从应用中发出的每种类型的邮件都代表了一个 "mailable" 的类。这些类被存储在 `app/Mail` 目录中。如果你的应用中并不存在这个目录，请不要担心，当你首次执行 `make:mail` 命令生成可以邮寄的类时，这个目录会自动的被创建：

    php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## 编写可邮寄的类

所有可邮寄类的配置都在 `build` 方法中完成。在这个方法中，你可以调用各种方法，比如 `from`，`subject`，`view` 和 `attach` 来进行邮件的展示与交付。

<a name="configuring-the-sender"></a>
### 配置发送者

#### 使用 `from` 方法

首先，让我们来探索一下邮件中关于发送者的配置。或者，换句话说，邮件是谁发出的。这里有两种方式来配置发送者。首先，你可以在可邮寄类的 `build` 方法中使用 `from` 方法：

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->view('emails.orders.shipped');
    }

#### 使用全局 `from` 地址

事实上，如果你的应用为所有的邮件提供相同的来源地址，那么在每个生成的可邮寄类中调用 `from` 方法会显得有些麻烦。那么，你可以在你的 `config/mail.php` 配置文件中指定一个全局 "from" 地址。如果再可邮寄的类中没有指定其它的 "from" 地址，那么它将使用这个全局的地址：

    'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

<a name="configuring-the-view"></a>
### 配置视图

在可邮寄类的 `build` 方法中，你可以使用 `view` 方法来为所渲染的邮件内容指定模板。正式由于邮件通常使用 [Blade 模板](/docs/{{version}}/blade) 来渲染它的内容，所有你可以借助强大的 Blade 模板引擎来非常方便的构建邮件的 HTML：

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }

> {tip} 你可能会希望创建一个 `resources/views/emails` 目录来存放所有的邮件模板。事实上，你可以自由的将其存放在 `resources/views` 目录下的任意位置。

#### 纯文本邮件

如果你希望为你的邮件定义纯文本的版本，那么你可以使用 `text` 方法。就像 `view` 方法一样，`text` 方法接收一个用于渲染邮件内容的模板名称，你可以自由的定义 HTML 和纯文本版本的消息：

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->text('emails.orders.shipped_plain');
    }

<a name="view-data"></a>
### 视图数据

#### 通过公开属性

通常，你会需要向视图传递一些数据将邮件渲染为 HTML。这里有两种方式可以使你在视图中访问这些数据。首先，在你的可邮寄类中的公开数据在视图中都是可以访问的。比如，你可以在可邮寄类的构造函数中传递一些数据，并将这些数据设置到类的公开属性中去：

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        public $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped');
        }
    }

当你的数据被设置为公开属性之后，它就可以在你的视图中进行访问。所以你就可以像访问其它数据一样在你的 Blade 模板中进行访问：

    <div>
        Price: {{ $order->price }}
    </div>

#### 通过 `with` 方法:

如果你希望在邮件的数据在发送到模板之前先进行自定义的格式化。那么通过使用 `with` 方法来手动的传递你的数据到视图中。通常的，你仍然需要在你的可邮寄类的构造函数中传递数据。但是，你应该设置这些数据为 `protected` 或者 `private` 属性，这样这些数据就不会被自动的传递到视图中去。然后，当你调用 `with` 方法时，你应该传递一个所期望的数组数据到你的模板中：

    <?php

    namespace App\Mail;

    use App\Order;
    use Illuminate\Bus\Queueable;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable
    {
        use Queueable, SerializesModels;

        /**
         * The order instance.
         *
         * @var Order
         */
        protected $order;

        /**
         * Create a new message instance.
         *
         * @return void
         */
        public function __construct(Order $order)
        {
            $this->order = $order;
        }

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->with([
                            'orderName' => $this->order->name,
                            'orderPrice' => $this->order->price,
                        ]);
        }
    }

当数据被传递到 `with` 方法之后，它会在动的被传递到视图中去，所以你可以像访问其它数据一样在你的 Blade 模板中访问它：

    <div>
        Price: {{ $orderPrice }}
    </div>

<a name="attachments"></a>
### 附件

你可以在可邮件类中的 `build` 方法中使用 `attach` 方法来为一个邮件附加一些附件。`attach` 方法接收文件的全路径作为它的第一个参数：

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file');
        }

当附加一个文件到消息时，你可能也需要指定附件显示的名称或者 MIME type。你可以向 `attach` 方法传递一个数组作为第二个参数：

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attach('/path/to/file', [
                            'as' => 'name.pdf',
                            'mime' => 'application/pdf',
                        ]);
        }

#### 附加原始数据

`attachData` 方法可以用来添加一个原始字节字符串作为一个附件。比如，你可能需要使用这个方法来将内存中生成的 PDF 直接添加到附件中，而不需要先存储到磁盘，`attachData` 方法接收原始字节数据作为它的第一个参数，附件的名称作为第二个参数，并且和一个选项数组作为第三个参数：

        /**
         * Build the message.
         *
         * @return $this
         */
        public function build()
        {
            return $this->view('emails.orders.shipped')
                        ->attachData($this->pdf, 'name.pdf', [
                            'mime' => 'application/pdf',
                        ]);
        }

<a name="inline-attachments"></a>
### 内联附件

在邮件中内联一个图片通常是比较笨重的。但是，Laravel 提供了一种便捷的方式在你的邮件中附加一个图片并且取回适当的 CID。你需要在邮件视图中使用 `$message` 变量的 `embed` 方法。你应该记得，Laravel 会自动的创建一个 `$message` 变量并将其传递到你的邮件视图中：

    <body>
        Here is an image:

        <img src="{{ $message->embed($pathToFile) }}">
    </body>

#### 嵌入原始数据到视图

如果你希望嵌入一个原始数据到邮件信息中，你可以使用 `$message` 变量的 `embedData` 方法：

    <body>
        Here is an image from raw data:

        <img src="{{ $message->embedData($data, $name) }}">
    </body>

<a name="sending-mail"></a>
## 发送邮件

你可以使用 `Mail` [假面](/docs/{{version}}/facades) 的 `to` 方法来发送一个邮件。`to` 放放接收邮件的地址，用户实例或者用户实例的集合作为参数。如果你传递一个对象或者一个集合对象，那么邮件在设置收件人时会自动的使用他们的 `emial` 和 `name` 属性，所以，你应该确保这些属性应该是可以在你的对象中访问的。当你指定完你的收件人之后，你就可以传递一个可邮寄类的实例到 `send` 方法中:

    <?php

    namespace App\Http\Controllers;

    use App\Order;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;
    use App\Http\Controllers\Controller;

    class OrderController extends Controller
    {
        /**
         * Ship the given order.
         *
         * @param  Request  $request
         * @param  int  $orderId
         * @return Response
         */
        public function ship(Request $request, $orderId)
        {
            $order = Order::findOrFail($orderId);

            // Ship order...

            Mail::to($request->user())->send(new OrderShipped($order));
        }
    }

当然，在发送消息时，你并没有被限制只能指定 `to` 到收件人，你也可以自由的组合使用 "to"，"cc"，"bcc" 到收件人，它们都是可以在一行代码中进行链式的调用：

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### 队列化邮件

#### 队列化一个邮件消息

由于发送邮件是非常消耗资源的一件事，这样会影响到应用的响应时间。所以很多开发者都选择使用队列来完成异步的邮件发送。Laravel 使用 [统一的队列接口](/docs/{{version}}/queues) 来使这些变的非常简单。你需要使用 `Mail` 假面的 `queue` 方法来使邮件队列化，它需要在指定完收件人之后调用:

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

该方法会自动的在队列添加发送邮件的任务，该任务会在后台自动的执行。当然，你需要在使用前先配置好 [队列](/docs/{{version}}/queues)。

#### 延迟消息队列

如果你希望延迟执行邮件的发送。那么你可以使用 `later` 方法。`later` 方法接收一个 `DateTime` 实例来指导邮件何时发送：

    $when = Carbon\Carbon::now()->addMinutes(10);

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later($when, new OrderShipped($order));

#### 添加到指定的队列

由于所有通过 `make:mail` 命令生成的可邮寄类都使用了 `Illuminate\Bus\Queueable` trait，所以你可以在任意的可邮寄类实例中调用 `onQueue` 和 `onConnection` 方法。这允许你为消息指定特殊的连接和队列:

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

<a name="mail-and-local-development"></a>
## 邮件 & 本地开发环境

当开发一个发送邮件的服务时，你可能并不希望真实的去发送一个邮件。只需要模拟测试成功就可以了。Laravel 为本地开发提供了多种方式来禁用真实的邮件发送。


#### Log 驱动

其中一个解决方案就是在本地开发时使用 `log` 邮件驱动。这个驱动会将所有的邮件信息写入到日志文件中。对于更多的关于应用环境配置的信息，请查看 [配置文档](/docs/{{version}}/installation#environment-configuration)

#### 通用的邮件

另外一个解决方案就是设置一个通用的邮件来接收所有的应用发出的邮件。这种方式，会使应用生成的所有邮件都发送到这个地址，而不是实际所指定的地址。你可以通过配置 `config/mail.php` 文件的 `to` 选项来进行设置：

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

最后，你也可以使用一些像 [Mailtrap](https://mailtrap.io/) 和 `smtp` 驱动的服务来发送你的邮件到一个虚拟邮箱，而且你也可以通过真实的邮件客户端进行查看。这可以使你看到真实的邮件在 Mailtrap 中的显示效果。

<a name="events"></a>
## 事件

Laravel 会在发送邮件之前触发一个事件。你应该注意的是，它是在邮件发送之前触发事件，而不是添加到队列时。你可以在 `EventServiceProvider` 中为该事件注册一个事件监听者：

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSentMessage',
        ],
    ];

