# 验证

- [前言](#introduction)
- [快速入门](#validation-quickstart)
    - [定义路由](#quick-defining-the-routes)
    - [创建控制器](#quick-creating-the-controller)
    - [编写验证逻辑](#quick-writing-the-validation-logic)
    - [显示验证错误](#quick-displaying-the-validation-errors)
- [表单请求验证](#form-request-validation)
    - [创建表单请求](#creating-form-requests)
    - [授权表单请求](#authorizing-form-requests)
    - [定制错误格式](#customizing-the-error-format)
    - [定制错误消息](#customizing-the-error-messages)
- [手动创建构造器](#manually-creating-validators)
    - [自动重定向](#automatic-redirection)
    - [命名错误袋](#named-error-bags)
    - [后置验证钩子](#after-validation-hook)
- [与错误消息交互](#working-with-error-messages)
    - [自定义错误消息](#custom-error-messages)
- [可用的验证规则](#available-validation-rules)
- [添加约束条件](#conditionally-adding-rules)
- [验证数组](#validating-arrays)
- [自定义验证约束](#custom-validation-rules)

<a name="introduction"></a>
## 前言

Laravel 对验证应用的输入数据提供了多中途径的实现。默认的，Laravel 的基础控制器类使用了 `ValidatesRequests` trait，该性状允许使用各种强大的验证约束来验证 HTTP 的输入请求。

<a name="validation-quickstart"></a>
## 快速入门

要了解 Laravel 强大的验证功能，我们需要一个完整的示例来描述表单的验证，和将表单验证的错误信息显示给用户。

<a name="quick-defining-the-routes"></a>
### 定义路由

首先，让我们假定我们在 `routes/web.php` 文件中拥有下述的路由：

    Route::get('post/create', 'PostController@create');

    Route::post('post', 'PostController@store');

当然， `GET` 路由会为用户创建一个新的博客文章时提供一个表单，而 `POST` 路由会存储新的博客文章到数据库。

<a name="quick-creating-the-controller"></a>
### 创建控制器

接着，我们需要一个控制器来处理这些路由，目前，我们先不在 `store` 方法里放任何的逻辑：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Show the form to create a new blog post.
         *
         * @return Response
         */
        public function create()
        {
            return view('post.create');
        }

        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            // Validate and store the blog post...
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### 编写验证逻辑

现在我们准备好了在 `store` 方法中进行博客文章的验证逻辑。如果你检查应用的基础控制器（`App\Http\Controllers\Controller`) 类，你会发现该类使用了 `ValidatesRequests` trait。这个性状为所有的控制器提供了方便的 `validate` 方法。

`validate` 方法接收 HTTP 输入请求，并设置验证约束。如果验证约束通过，那么后续的代码将会正常的执行。如果验证失败，那么将会抛出一个恰当的异常响应返回给用户。对于传统的 HTTP 请求，验证器会自动生成一个重定向响应，而 AJAX 请求，则会返回 JSON 响应。

为了能够更好的理解 `validate` 方法，让我们继续回到 `store` 方法：

    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $this->validate($request, [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // The blog post is valid, store in database...
    }

就如你所看到的，我们简单的传递了一个 HTTP 输入请求，并且在 `validate` 方法中设置了预期的验证约束。而这次，如果验证失败，那么相应的响应会被自动的生成并且被返回给请求用户。如果验证通过，那么我们的控制器会继续执行之后的业务。

#### 在初次验证失败时停止

有时候你希望在获取首个验证约束失败时停止当前属性其余约束的验证。你可以在属性中加入 `bail` 约束：

    $this->validate($request, [
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

在这个例子中，如果 `title` 属性中的 `required` 约束验证失败，那么 `unique` 约束就不会再被验证。约束是按照其被分配的顺序来进行验证的。

#### 嵌套的属性

如果你的 HTTP 请求包含了嵌套的参数，你可以使用 `.` 语法来为其指定约束：

    $this->validate($request, [
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### 显示验证错误

那么，假如传入的请求参数并没有通过给定约束的验证怎么办？就如前面所提到的，Laravel 会自动的重定向用户到之前的位置。另外，所有的验证错误信息都会被自动的闪存到 [session](/{{language}}/{{version}}/session#flash-data) 中。

你需要注意到我们并没有明确的绑定错误信息到 `GET` 路由的响应视图里。这是因为 laravel 会检查闪存 seesion 里的错误数据，并且会自动的在其可用时注入到视图中。你可以在视图中使用 `$errors` 变量，它是一个 `Illuminate\Support\MessageBag` 实例。如果需要了解更多这个实例对象，请参考其 [文档](#working-with-error-messages)。

> {tip} `$errors` 变量是通过 `Illuminate\View\Middleware\ShareErrorsFromSession` 中间件来绑定到视图中的。这个中间件已经被提供到了 `web` 中间件组中。**这个中间件被应用时会自主的在你的视图中注入 `$errors` 变量**，这允许你方便的假定 `$errors` 变量总是已经被定义且可以安全的使用。

所以，在我们的例子中，当验证失败是，用户将会被重定向到控制器的 `create` 方法中，这允许我们在视图中展示错误信息：

    <!-- /resources/views/post/create.blade.php -->

    <h1>Create Post</h1>

    @if (count($errors) > 0)
        <div class="alert alert-danger">
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <!-- Create Post Form -->

<a name="quick-customizing-the-flashed-error-format"></a>
#### 自定义闪存错误格式

如果你希望在验证失败时可以自定义闪存进 session 中的错误消息的格式，你需要在你的基础控制器中复写 `formatValidationErrors` 方法。不要忘记在顶部引入 `Illuminate\Contracts\Validation\Validator` 类：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Foundation\Bus\DispatchesJobs;
    use Illuminate\Contracts\Validation\Validator;
    use Illuminate\Routing\Controller as BaseController;
    use Illuminate\Foundation\Validation\ValidatesRequests;

    abstract class Controller extends BaseController
    {
        use DispatchesJobs, ValidatesRequests;

        /**
         * {@inheritdoc}
         */
        protected function formatValidationErrors(Validator $validator)
        {
            return $validator->errors()->all();
        }
    }

<a name="quick-ajax-requests-and-validation"></a>
#### AJAX 请求 & 验证

在上面的示例中，我们使用传统的表单来发送数据到应用，事实上，如今很多应用都使用 AJAX 请求，当通过 AJAX 请求来使用 `validate` 方法时，Laravel 并不会自动生成重定向的响应，相反的，Laravel 会生成一个包含了验证错误消息的 JSON 响应。并且该响应会伴随 422 HTTP 状态码。

<a name="form-request-validation"></a>
## 表单请求验证

<a name="creating-form-requests"></a>
### 创建表单请求

对于更为复杂的验证场景，你或许希望构建一个“表单请求”。表单请求是一个自定义的请求类，并且它包含了所有的验证逻辑。你可以使用 `make:request` Artisan CLI 命令来创建一个表单请求类：

    php artisan make:request StoreBlogPost

被生成的类会被存储在 `app/Http/Requests` 目录。如果这个目录并不存在，那么它会在你执行 `make:request` 命令之后生成。让我们在 `rules` 方法中来添加一些验证约束：

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

那么，这些验证约束是如何被评定的？你所要做的所有的事情就是在你的控制器方法中添加该请求类的类型提示。传入进来的表单请求会在控制器方法调用之前被自动的进行约束验证，这意味着你完全不需要在你的控制器方法中添加任何乱糟糟的验证逻辑：

    /**
     * Store the incoming blog post.
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // The incoming request is valid...
    }

如果验证失败，用户会被自动的重定向到他们之前的位置。那么验证错误消息也会自动的闪存进 session 数据中被用于显示。如果你使用的是 AJAX 请求，那么会自动的返回一个包含所有验证错误消息的 JSON 格式的响应，它的 HTTP 状态码会被设置为 422。

<a name="authorizing-form-requests"></a>
### 授权表单请求

表单请求类也包含了 `authorize` 方法。在这个方法中，你可以检查已认证的用户是否真的拥有修改所给定资源的权利。比如，如果用户尝试修改博客文章中的评论消息，我们需要考虑一下这个评论是属于他的吗：

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        $comment = Comment::find($this->route('comment'));

        return $comment && $this->user()->can('update', $comment);
    }

由于所有的表单请求类都继承自底层的 Laravel 请求类，所有我们可以使用 `user` 方法来访问当前认证的用户。你应该注意到了上述实例中的 `route` 方法的调用。这个方法用来在路由被访问时发放所定义的 URL 参数，比如下面路由的 `{comment}` 参数:

    Route::post('comment/{comment}');

如果 `authorize` 方法返回 `false`，那么会响应一个 403 的状态码，并且控制器的方法不会被执行。

如果你计划在应用的其它部分来处理授权逻辑，你可以简单的在 `authorize` 方法中返回 `true`:

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

<a name="customizing-the-error-format"></a>
### 定制错误格式

如果你希望在验证失败时自定义闪存到 session 数据中验证错误消息的格式，那么你需要复写（`App\Http\Requests\Request`）基础请求类中的 `formatErros` 方法。不要忘记引入 `Illuminate\Contracts\Validation\Validator` 类：

    /**
     * {@inheritdoc}
     */
    protected function formatErrors(Validator $validator)
    {
        return $validator->errors()->all();
    }

<a name="customizing-the-error-messages"></a>
### 定制错误消息

你也可以通过在请求类中复写 `messages` 方法来自定义错误消息。该方法应该返回一个包含相应错误消息的键值对数组：

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => 'A title is required',
            'body.required'  => 'A message is required',
        ];
    }

<a name="manually-creating-validators"></a>
## 手动的创建验证器

如果你不喜欢使用 `ValidatesRequests` trait 的 `validator` 方法，你也可以通过使用 `Validator` [假面](/{{language}}/{{version}}/facades) 来创建一个 validator 实例。`Validator` 假面的 `make` 方法就可以生成一个新的 validator 实例：

    <?php

    namespace App\Http\Controllers;

    use Validator;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class PostController extends Controller
    {
        /**
         * Store a new blog post.
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // Store the blog post...
        }
    }

`make` 方法所接受的第一个参数是需要被验证的数据，第二个参数则是应该施加到数据的验证约束。

如果请求的验证失败，那么你需要使用 `withErrors` 方法来将错误消息存放到 session 中。当使用该方法时，`$errors` 变量会在重定向之后被自动的共享到你的视图中，这使你可以轻松的将错误信息展示给用户。`withErrors` 方法可以接收 validator 实例，或者 `MessageBag` 实例，又或者原生的 PHP `array`。

<a name="automatic-redirection"></a>
### 自动重定向

如果你喜欢手动的创建一个验证器但是你仍然希望使用 `ValidatesRequest` trait 所提供的自动重定向的特性。那么你可以调用验证器实例的 `validate` 方法。如果验证失败，那么用户将会被自动的重定向，在 AJAX 请求场景中，则会返回 JSON 类型的响应:

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

<a name="named-error-bags"></a>
### 命名的错误袋

如果你在一个独立页面中包含了多个表单。那么你可能会希望能对 `MessageBag` 进行命名以展示相应的表单错误。你可以直接在 `withErrors` 方法中传递第二个参数对其进行命名:

    return redirect('register')
                ->withErrors($validator, 'login');

你之后可以通过 `$errors` 变量来访问被命名的 `MessageBag` 实例：

    {{ $errors->login->first('email') }}

<a name="after-validation-hook"></a>
### 后置验证钩子

验证器也允许你在验证完成之后执行特定的操作。这允许你轻松的进行进一步的验证，你甚至可以在消息集合里添加更多的错误消息。你可以在验证器的实例上使用 `after` 方法来进行 hook:

    $validator = Validator::make(...);

    $validator->after(function($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });

    if ($validator->fails()) {
        //
    }

<a name="working-with-error-messages"></a>
## 与错误消息交互

在调用 `Validator` 实例的 `errors` 方法之后，你可以检索到一个 `Illuminate\Support\MessageBag` 的实例，该实例拥有多种便捷的方法来与错误消息进行交互。其中 `$errors` 变量既是 `MessageBag` 类的实例，又可以在所有的视图进行访问。

#### 检索字段中的首个错误消息

你可以使用 `first` 方法来检索给定字段的首个错误消息：

    $errors = $validator->errors();

    echo $errors->first('email');

#### 检索字段的所有错误消息

如果你需要检索给定字段的所有消息所组成的数组，那么你应该使用 `get` 方法：

    foreach ($errors->get('email') as $message) {
        //
    }

如果你在验证一个数组表单字段，你可以使用 `*` 字符来检索每个数组元素中的所有消息:

    foreach ($errors->get('attachments.*') as $message) {
        //
    }

#### Retrieving All Error Messages For All Fields

你可以使用 `all` 方法来检索所有字段的所有错误消息所组成的数组：

    foreach ($errors->all() as $message) {
        //
    }

#### 判断所给定的字段中是否存在消息

`has` 方法可以用来判断所给定的字段中是否存在任意错误消息:

    if ($errors->has('email')) {
        //
    }

<a name="custom-error-messages"></a>
### 自定义错误消息

如果你需要，你可以使用自定义的错误消息来取代默认的验证消息。这里有几种方式来指定自定义的消息。首先，你可以传递自定的消息作为 `Validator::make` 方法的第三个参数：

    $messages = [
        'required' => 'The :attribute field is required.',
    ];

    $validator = Validator::make($input, $rules, $messages);

在这个例子中，`:attribute` 占位符会被验证数据中真实的名称所替换。你还可以利用其它的占位符到验证消息中，比如：

    $messages = [
        'same'    => 'The :attribute and :other must match.',
        'size'    => 'The :attribute must be exactly :size.',
        'between' => 'The :attribute must be between :min - :max.',
        'in'      => 'The :attribute must be one of the following types: :values',
    ];

#### 为给定的属性指定错误消息

有时候，你可能希望指定自定义的错误消息到特定的字段。你可以使用 `.` 符号来进行分割，属性名应该在前，约束应该在后：

    $messages = [
        'email.required' => 'We need to know your e-mail address!',
    ];

<a name="localization"></a>
#### 在语言文件中指定自定义消息

在多数情况下，你可能希望使用一个语言文件中的自定义消息属性直接传递到 `Validator`。你可以在 `resources/lang/xx/validation.php` 语言文件中添加 `custom` 数组来存储你的消息：

    'custom' => [
        'email' => [
            'required' => 'We need to know your e-mail address!',
        ],
    ],

<a name="available-validation-rules"></a>
## 可用的验证约束

下面是所有的可用的验证约束和它们的功能的列表：

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Before (Date)](#rule-before)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[E-Mail](#rule-email)
[Exists (Database)](#rule-exists)
[File](#rule-file)
[Filled](#rule-filled)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Nullable](#rule-nullable)
[Not In](#rule-not-in)
[Numeric](#rule-numeric)
[Present](#rule-present)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)

</div>

<a name="rule-accepted"></a>
#### accepted

验证的字段必须为 *yes*，*on*，1，或者 *true*。这通常用来验证服务条款的承诺。

<a name="rule-active-url"></a>
#### active_url

验证的字段必须可以通过 `checkdnsrr` PHP 方法的验证。

<a name="rule-after"></a>
#### after:_date_

验证的字段必须是给定日期之后的值。日期会被传递到 `strtotime` PHP 方法：

    'start_date' => 'required|date|after:tomorrow'

你也可以指定使用其它字段的日期来进行评估：

    'finish_date' => 'required|date|after:start_date'

<a name="rule-alpha"></a>
#### alpha

验证的字段必须全部是由字母字符组成的字符串。

<a name="rule-alpha-dash"></a>
#### alpha_dash

验证的字段可以是字母，数字，-，_ 所组成的字符串。

<a name="rule-alpha-num"></a>
#### alpha_num

验证的字段必须全部由字母或数字所组成。

<a name="rule-array"></a>
#### array

验证的字段必须是一个 PHP `array`。

<a name="rule-before"></a>
#### before:_date_

验证的字段的值必须比指定的日期要早。指定的日期会被传递到 PHP 的 `strtotime` 方法。

<a name="rule-between"></a>
#### between:_min_,_max_

验证的字段的大小必须在给定的 *min* 和 *max* 之间。字符串，数字和文件都会使用和 [`size`](#rule-size) 约束的相同的评估方式。

<a name="rule-boolean"></a>
#### boolean

验证的字段必须能够转换为布尔值。所接受的输入可以是 `true`，`false`，`1`，`0`，`"1"`，`"0"`。

<a name="rule-confirmed"></a>
#### confirmed

验证的字段必须能够和 `foo_confirmation` 字段相匹配。比如，如果验证的字段是 `password`，相应的 `password_confirmation` 字段必须在输入中被提供且与 `password` 相匹配。

<a name="rule-date"></a>
#### date

验证的字段必须是一个有效的日期，它应该能被 `strtotime` PHP 方法通过。

<a name="rule-date-format"></a>
#### date_format:_format_

验证的字段必须匹配给定的格式。该格式会被 PHP `date_parse_from_format` 方法评定，你应该只使用 `date` 或者 `date_format` 其中之一来进行验证字段，不要全部都使用。

<a name="rule-different"></a>
#### different:_field_

验证的字段必须与给定的字段不同。

<a name="rule-digits"></a>
#### digits:_value_

验证的字段必须是数字类型并且具有指定的长度。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

验证的字段必须具有指定区间的长度。

<a name="rule-dimensions"></a>
#### dimensions

验证的字段必须是一个图片类型的，并且要求符合指定的参数约束:

    'avatar' => 'dimensions:min_width=100,min_height=200'

可用的参数有：*min_width*，*max_width*，*min_height*，*max_height*，*width*，*height*，*ratio*。

**ratio** 应该表示宽度除以身高。它应该被指定为 `3/2` 的声明或者 `1.5` 类似的浮点数。

    'avatar' => 'dimensions:ratio=3/2'

<a name="rule-distinct"></a>
#### distinct

当与数组协作时，验证的字段中必须不能含有重复的值：

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

验证的字段必须是一个邮件地址的格式。

<a name="rule-exists"></a>
#### exists:_table_,_column_

验证的字段必须能在指定数据库表中检索的到。

#### Exists 基础约束用法

    'state' => 'exists:states'

#### 指定自定义列名称

    'state' => 'exists:states,abbreviation'

你也可以像使用 `where` 语句一样指定添加更多的查询条件：

    'email' => 'exists:staff,email,account_id,1'

查询条件也可以使用 `!` 来表明否定值:

    'email' => 'exists:staff,email,role,!admin'

你也可以传递 `NULL` 或者 `NOT_NULL` 到查询语句中：

    'email' => 'exists:staff,email,deleted_at,NULL'

    'email' => 'exists:staff,email,deleted_at,NOT_NULL'

极个别的情况下，你可能需要在 `exists` 查询下指定特定的数据库连接。你可以使用 `.` 语法将数据库连接名前置来进行指定：

    'email' => 'exists:connection.staff,email'

<a name="rule-file"></a>
#### file

验证的字段必须是一个成功上传的文件。

<a name="rule-filled"></a>
#### filled

验证的字段如果出现，那么它一定不能为空值。

<a name="rule-image"></a>
#### image

被验证的文件必须是一个图片类型（jpeg，png，bmp，gif，svg）

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

验证的字段必须是给定值列中的一个。

<a name="rule-in-array"></a>
#### in_array:_anotherfield_

验证的字段必须是指定的字段中值列之一。

<a name="rule-integer"></a>
#### integer

验证的字段必须是一个整数。

<a name="rule-ip"></a>
#### ip

验证的字段必须是一个 IP 地址。

<a name="rule-json"></a>
#### json

验证的字段必须是合法的 JSON 字符串

<a name="rule-max"></a>
#### max:_value_

验证的字段必须小于等于指定值。字符串，数字，和文件类型会与 [`size`](#rule-size) 约束使用相同的评估方法。

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

验证的字段必须匹配给定的 MIME 类型：

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

为了判断所上传文件的 MIME 类型，Laravel 会读取文件的内容并且会尝试猜测文件的 MIME 类型，这可能会与客户端提供的文件 MIME 类型有所区别。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

验证的文件的 MIME 类型相应的后缀必须是所列的值之一。

#### MIME 约束基础用法

    'photo' => 'mimes:jpeg,bmp,png'

即使你指定了文件的扩展，这个约束也会针对文件的内容进行猜测文件的 MIME 类型，然后进行扩展验证。

完整的 MIME 类型和其相应的扩展后缀，你可以从 [这里](http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types) 找到。

<a name="rule-min"></a>
#### min:_value_

验证的字段必须比指定的值要小。字符串，数字和文件类型会使用 [`size`](#rule-size) 约束相同的评估方法。

<a name="rule-nullable"></a>
#### nullable

被验证的字段可以为 `null`。这个常用于验证那些可以包含 `null` 值的原始类型,如字符串和整数。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

验证的字段不应该包含在给定的值列中。

<a name="rule-numeric"></a>
#### numeric

验证的字段必须是一个数值类型。

<a name="rule-present"></a>
#### present

验证的字段必须要求被提供，但是可以为空。

<a name="rule-regex"></a>
#### regex:_pattern_

验证的字段必须与给定的正则表达式相匹配。

> {tip} 当使用 `regex` 模式时，你必须将约束放进数组里来取代管道符分隔，特别是在正则表达式中包含管道符时。

<a name="rule-required"></a>
#### required

验证的字段必须被提供并且不能为空值。判断空值的依据：

<div class="content-list" markdown="1">
- 值是 `null`
- 值是空字符串
- 值是一个空数组或者空的 `Countable` 对象
- 值是一个没有传递路径的上传文件
</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

验证的字段必须在以下情况下被提供：指定的字段等于任意列出的值。

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

验证的字段必须在以下情况下被提供: 所指定的字段和所提供的值都不相等。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

验证的字段只有在其它所指定字段之一被提供时才会被要求提供。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

验证的字段只有在其它所指定字段全部被提供时才会被要求提供。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

验证的字段只有在其它所指定字段之一没有被提供时被要求提供。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

验证的字段只有在所指定字段全部没有被提供时才会被要求提供。

<a name="rule-same"></a>
#### same:_field_

所验证的字段必须与指定的字段相匹配。

<a name="rule-size"></a>
#### size:_value_

验证的字段必须具有给定值的大小。对于字符串数据，值应该是字符串的字符长度。对于数值数据，值应该是相应的整数值。对于数组，大小匹配数组的 `count` 大小。对于文件，应该匹配文件的字节大小。

<a name="rule-string"></a>
#### string

验证的字段必须是一个字符串。如果你也喜欢允许这个字段为 `null`，那么你应该分配 `nullable` 约束到这个字段。

<a name="rule-timezone"></a>
#### timezone

验证的字段必须是经过 PHP `timezone_identifiers_list` 方法验证的合法的 timezone 标识。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

验证的字段必须在给定的数据表中唯一，如果 `column` 选型没有被指定，那么会直接使用字段的名字。

**指定自定义的列名**

    'email' => 'unique:users,email_address'

**自定义数据库连接**

极少数情况下，你可能需要指定自定义的数据库连接来进行验证。就如上面所看到的，设置 `unique:users` 会使用默认的数据库连接来进行约束验证。如果想指定其他数据库连接，你可以使用 `.` 语法并前置指定数据库连接：

    'email' => 'unique:connection.users,email_address'

**强迫 Unique 约束 忽略给定的 ID**

有时候，你可能会希望 unique 检查忽略给定的 ID。比如，考虑一下一个更新个人信息的场景，它应该提供用户的名称，邮箱地址，和位置。你可能会想要验证邮箱的唯一性。但是你只想验证与用户的当前邮箱不一致的邮箱的唯一性。也就是说你只想验证这个邮箱有没有被其他用户所使用。你需要传递 ID 作为第三个参数来通知 unique 约束来忽略当前用户的 ID：

    'email' => 'unique:users,email_address,'.$user->id

如果表名使用的主键列名不是 `id` ，那么你还需要指定主键的列名到第四个参数：

    'email' => 'unique:users,email_address,'.$user->id.',user_id'

**添加额外的条件查询**

你也可以指定更多的条件查询：

    'email' => 'unique:users,email_address,NULL,id,account_id,1'

在上面的约束中，只有 `account_id` 为 `1` 的行会被约束进行检查。

<a name="rule-url"></a>
#### url

验证的字段必是一个有效 URL。

<a name="conditionally-adding-rules"></a>
## 添加约束条件

#### 当存在时验证

在一些场景中，你会希望只有字段出现在了输入数组中时才会对其进行验证。你可以在约束列中添加 `sometimes` 约束来快速的完成指定：

    $v = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

这上面的例子中，只有 `$data` 中提供了 `email` 字段，`email` 的约束才会对其进行验证。

#### 复杂的验证条件

有时候，你可能会希望基于更复杂的条件逻辑去进行约束的验证。比如，你希望只有另外一个字段拥有比 100 更大的值时才会验证给定的字段是否被提供。又或者你想要在只有另外一个字段被提供时才会需要其他两个字段的值。添加这些条件判定并非是痛苦的一件事。首先，你还是需要创建一个 `Validator` 实例和一些静态的约束：

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

让我们假定我们的应用是服务于一些游戏收藏家的。如果一个游戏收藏家注册了我们的应用，并且它们添加了超过 100 个游戏时。我们需要它们解释一下为什么他会拥有那么多的游戏。比如，或许他开了一个游戏贩卖超市，又或者他仅仅就是喜欢收藏。我们可以使用 `Validator` 实例上的 `sometimes` 方法来添加这个必要的条件：

    $v->sometimes('reason', 'required|max:500', function($input) {
        return $input->games >= 100;
    });

传递到 `sometimes` 方法的第一个参数是我们需要考虑验证的字段的名字。第二个参数是我们想要添加的约束。如果第三个参数传递的 `Closure` 返回的结果是 `true`，那么这个约束就会被添加进去。这就可以轻松的对复杂的验证场景进行条件的构建。你甚至可以一次性的添加多个字段的条件验证：

    $v->sometimes(['reason', 'cost'], 'required', function($input) {
        return $input->games >= 100;
    });

> {tip} 传递到 `Closure` 中的 `$input` 是一个 `Illuminate\Support\Flument` 实例，并且它可以被用来访问你的输入和文件。

<a name="validating-arrays"></a>
## Validating Arrays

验证数组形式的输入并不是一件痛苦的事情。比如，去验证给定的输入数组中所有的邮件都应该是唯一的，你可以参照如下做法：

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

同样的，你也可以在使用语言文件来指定特定的验证消息时使用 `*` 通配符。这可以轻而易举的使用单条验证消息提供给基于数组的输入：

    'custom' => [
        'person.*.email' => [
            'unique' => 'Each person must have a unique e-mail address',
        ]
    ],

<a name="custom-validation-rules"></a>
## Custom Validation Rules

Laravel 提供了各种有用的验证约束。但是，你可能希望添加你自己的特定的约束。你可以使用 `Validator` [假面](/{{language}}/{{version}}/facades) 的 `extend` 方法来注册自己的验证约束。让我们在 [服务提供者](/{{language}}/{{version}}/providers) 里注册一个自定义的验证约束：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Support\Facades\Validator;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            Validator::extend('foo', function($attribute, $value, $parameters, $validator) {
                return $value == 'foo';
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

自定义的验证闭包中接收四个参数：需要被验证的属性名称，属性的值，一个需要被传递到约束的 `$paramters` 数组，和 `Validator` 实例。

你也可以传递一个类名和方法到 `extend` 方法中来代替闭包：

    Validator::extend('foo', 'FooValidator@validate');

#### 定义错误消息

你也需要为你的自定义约束添加一个错误消息。你可以使用行内自定义错误消息或者将其添加到独立的验证语言文件中。这个消息应该被存放在数组的一维中，而不是包含在 `custom` 数组里:

    "foo" => "Your input was invalid!",

    "accepted" => "The :attribute must be accepted.",

    // The rest of the validation error messages...

当构建自定义的验证约束时，你或许有时候也想为错误消息定义一些占位符。你可以使用 `Validator` 假面的 `replacer` 方法来进行占位替换。你可以在 [服务提供者](/{{language}}/{{version}}/providers) 的 `boot` 方法中来做这些：

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Validator::extend(...);

        Validator::replacer('foo', function($message, $attribute, $rule, $parameters) {
            return str_replace(...);
        });
    }

#### 隐式扩展

默认的，当属性被验证时，如果在输入数组中没有被提供，或者验证约束为 [`required`](#rule-required) 却是一个空值。那么普通的验证约束，包括自定义的约束扩展，都不会再执行。比如，`name` 属性的 [`unique`](#rule-unique) 约束就不会在 `name` 为 `null` 值时执行：

    $rules = ['name' => 'unique'];

    $input = ['name' => null];

    Validator::make($input, $rules)->passes(); // true

如果需要约束即使是属性值为空时也继续执行，那么约束需要暗示属性是必须的。你可以使用 `Validator::extendImplicit()` 方法来构建一个 “隐式的” 扩展：

    Validator::extendImplicit('foo', function($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });

> {note} 隐式的扩展仅仅是暗示属性是必须的，不论它实际上是缺失的值或者是空的属性，这都取决于你。
