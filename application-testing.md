# 应用测试

- [前言](#introduction)
- [与应用交互](#interacting-with-your-application)
    - [与连接交互](#interacting-with-links)
    - [与表单交互](#interacting-with-forms)
- [测试 JSON APIs](#testing-json-apis)
    - [完全匹配验证](#verifying-exact-match)
    - [结构匹配验证](#verifying-structural-match)
- [会话 / 认证](#sessions-and-authentication)
- [禁用中间件](#disabling-middleware)
- [自定义 HTTP 请求](#custom-http-requests)
- [PHPUnit 断言](#phpunit-assertions)

<a name="introduction"></a>
## 前言

Laravel 可以提供非常顺畅的 API 来构建 HTTP 请求用以检查输出甚至是填充表格。比如，让我们看下下面的测试定义：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5')
                 ->dontSee('Rails');
        }
    }

`visit` 方法可以在应用中构造一个 `GET` 请求。`see` 方法会断言我们应该从应用的响应中看到所给定的文本。`dontSee` 方法刚好与 `see` 相反，它断言我们不应该看到给定的文本。这就是 Laravel 提供的最基本的应用测试。

你也可以使用 `visitRoute` 方法通过命名的路由构造一个 `GET` 请求：

    $this->visitRoute('profile');

    $this->visitRoute('profile', ['user' => 1]);

<a name="interacting-with-your-application"></a>
## 与应用进行交互

当然，相对于这种简单的断言响应给定的文本，你可以做的更多。让我们来看一些点击链接和填充表单的例子：

<a name="interacting-with-links"></a>
### 与链接交互

在这个测试，我们会为应用构造一个请求，在响应中会返回一个可点击的超链，并且我们会断言它应该导向给定的 URL。比如，让我们假设应用中返回了一个文本为 "About Us" 的超链:

    <a href="/about-us">About Us</a>

现在，让我们编写一个测试并断言点击链接会跳转到相应的页面：

    public function testBasicExample()
    {
        $this->visit('/')
             ->click('About Us')
             ->seePageIs('/about-us');
    }

你也可以使用 `seeRouteIs` 方法来检查用户是否抵达正确的命名路由：

    ->seeRouteIs('profile', ['user' => 1]);

<a name="interacting-with-forms"></a>
### 与表单交互

Laravel 同样也提供了多种方法来进行表单测试。`type`，`select`，`check`，`attach`，和 `press` 方法允许你与所有的表单输入进行交互。比如，让我们来想象一下一个存在于注册页面中的表单：

    <form action="/register" method="POST">
        {{ csrf_field() }}

        <div>
            Name: <input type="text" name="name">
        </div>

        <div>
            <input type="checkbox" value="yes" name="terms"> Accept Terms
        </div>

        <div>
            <input type="submit" value="Register">
        </div>
    </form>

我们可以编写一个测试来完成表单并检查结果：

    public function testNewUserRegistration()
    {
        $this->visit('/register')
             ->type('Taylor', 'name')
             ->check('terms')
             ->press('Register')
             ->seePageIs('/dashboard');
    }

当然，如果你的表单中包含了一些单选按钮或者下拉框之类的其他输入，你同样可以轻松的进行填充这些类型的字段。下面列出了所有的表单操作方法：

Method  | Description
------------- | -------------
`$this->type($text, $elementName)`         | 对给定的字段进行输入
`$this->select($value, $elementName)`      | 选中一个单选按钮或者下拉框
`$this->check($elementName)`               | 选中复选框
`$this->uncheck($elementName)`             | 取消选中复选框
`$this->attach($pathToFile, $elementName)` | 附加一个文件到表单
`$this->press($buttonTextOrElementName)`   | 单击给定的文本或名称的按钮

<a name="file-inputs"></a>
#### 文件输入

如果你的表单中包含了 `file` 输入类型，你可以使用 `attach` 方法来附加附件：

    public function testPhotoCanBeUploaded()
    {
        $this->visit('/upload')
             ->attach($pathToFile, 'photo')
             ->press('Upload')
             ->see('Upload Successful!');
    }

<a name="testing-json-apis"></a>
### 测试 JSON APIs

Laravel 同样也为测试 JSON APIs 和它们的响应提供了多种帮助方法。比如，`get`，`post`，`put`，`patch`，和 `delete` 方法可以用来发布一个相应 HTTP 行为方式的请求。你可以轻松的传递数据和头信息到这些方法中。在开始之前，让我们来编写一个 `POST` 请求的测试来断言 `/user` 会返回给定数组的 JSON 格式：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->json('POST', '/user', ['name' => 'Sally'])
                 ->seeJson([
                     'created' => true,
                 ]);
        }
    }

> {tip} `seeJson` 方法会转换给定的数组为 JSON，并且会验证应用响应的完整 JSON 中是否会出现相应的片段。所以，如果响应中还含有其他 JSON 属性，那么这个测试依然会被通过。

<a name="verifying-exact-match"></a>
### 完全匹配验证

如果你希望验证完整的 JSON 响应，你可以使用 `seeJsonEquals` 方法，除非 JSON 相应于所给定的数组完全匹配，否则该测试不会被通过：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->json('POST', '/user', ['name' => 'Sally'])
                 ->seeJsonEquals([
                     'created' => true,
                 ]);
        }
    }

<a name="verifying-structural-match"></a>
### 结构匹配验证

验证 JSON 响应是否采取给定的结构也是可以的。你可以使用 `seeJsonStructure` 方法并传递你所期待的 JSON 结构：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->get('/user/1')
                 ->seeJsonStructure([
                     'name',
                     'pet' => [
                         'name', 'age'
                     ]
                 ]);
        }
    }

在上面的例子中表明了期望获取一个含有 `name` 和 `pet` 属性的 JSON，并且 `pet` 键是一个含有 `name` 和 `age` 属性的对象。如果含有额外的键，`seeJsonStructure` 方法并不会失败。比如，如果 `pet` 还含有 `weight` 属性，那么测试依然会被通过。

你可以使用 `*` 来断言所返回的 JSON 结构中的每一项都应该包含所列出的这些属性：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            // Assert that each user in the list has at least an id, name and email attribute.
            $this->get('/users')
                 ->seeJsonStructure([
                     '*' => [
                         'id', 'name', 'email'
                     ]
                 ]);
        }
    }

你也可以嵌套使用 `*`，下面的例子中，我们断言 JSON 响应返回的每一个用户都应该包含所列出的属性，并且 `pet` 属性应该也包含所给定的属性：

    $this->get('/users')
         ->seeJsonStructure([
             '*' => [
                 'id', 'name', 'email', 'pets' => [
                     '*' => [
                         'name', 'age'
                     ]
                 ]
             ]
         ]);

<a name="sessions-and-authentication"></a>
### 会话 / 认证

Laravel 为测试期间的会话提供了多种帮助方法。首先，你需要通过 `withSession` 方法来根据给定的数组设置 session 数据，这通常用来在测试应用的请求到在之前先设置一些 session 数据：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $this->withSession(['foo' => 'bar'])
                 ->visit('/');
        }
    }

当然，会话常见的用途就是保留用户的状态，比如用户认证。`actingAs` 帮助方法提供了一种方式来使用给定的用户作为已认证的当前用户。比如，我们可以使用 [模型工厂](/{{language}}/{{version}}/database-testing#model-factories) 来生成一个认证用户：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory(App\User::class)->create();

            $this->actingAs($user)
                 ->withSession(['foo' => 'bar'])
                 ->visit('/')
                 ->see('Hello, '.$user->name);
        }
    }

你也可以传递一个给定的守卫名称到 `actingAs` 方法的第二个参数来指定用户认证时所使用的守卫：

    $this->actingAs($user, 'api')

<a name="disabling-middleware"></a>
### 禁用中间件

当测试应用时，你可以方便的在测试中禁用 [中间件](/{{language}}/{{version}}/middleware)。这使你可以隔离的测试路由和控制器而免除中间件的顾虑。你可以简单的引入 `WithoutMiddleware` trait 来在测试类中禁用所有的中间件：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use WithoutMiddleware;

        //
    }

如果你希望只在个别的测试方法中禁用中间件，你可以在方法中使用 `withoutMiddleware` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->withoutMiddleware();

            $this->visit('/')
                 ->see('Laravel 5');
        }
    }

<a name="custom-http-requests"></a>
### 定制化 HTTP 请求

如果你希望构造一个自定义的 HTTP 请求，并且返回完整的 `Illuminate\Http\Response` 对象，那么你可以使用 `call` 方法：

    public function testApplication()
    {
        $response = $this->call('GET', '/');

        $this->assertEquals(200, $response->status());
    }

如果你构造 `POST`，`PUT`，或者 `PATCH` 请求，你可以传递一个数组来作为请求的输入数据。当然，这些数据可以通过 [请求实例](/{{language}}/{{version}}/requests) 在你的路由和控制器中可用：

       $response = $this->call('POST', '/user', ['name' => 'Taylor']);

<a name="phpunit-assertions"></a>
### PHPUnit 断言

Laravel 为 [PHPUnit](https://phpunit.de/) 测试提供了多种额外的断言方法：

Method  | Description
------------- | -------------
`->assertResponseOk();`                                           | 断言客户端响应会返回 OK 状态码
`->assertResponseStatus($code)`                                   | 断言客户端响应给定的状态码
`->assertViewHas($key, $value = null);`                           | 断言响应的视图中是否含有给定的数据片段。
`->assertViewHasAll(array $bindings);`                            | 断言视图中是否含有所有的绑定数据
`->assertViewMissing($key);`                                      | 断言视图中是否缺失所给定的片段
`->assertRedirectedTo($uri, $with = []);`                         | 断言客户端是否重定向到给定的 URI
`->assertRedirectedToRoute($name, $parameters = [], $with = []);` | 断言客户端是否重定向到给定的路由
`->assertRedirectedToAction($name, $parameters = [], $with = []);`  | 断言客户端是否重定向到给定的动作
`->assertSessionHas($key, $value = null);`                        | 断言会话中是否含有给定的键
`->assertSessionHasAll(array $bindings);`                         | 断言会话中是否包含给定列表的数据
`->assertSessionHasErrors($bindings = [], $format = null);`       | 断言会话中是否包含错误数据
`->assertHasOldInput();`                                          | 断言会话中是否包含旧的输入
`->assertSessionMissing($key);`                                   | 断言会话中是否缺失给定的键
