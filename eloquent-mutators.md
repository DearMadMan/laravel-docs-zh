# Eloquent: 调节器

- [前言](#introduction)
- [访问器 & 存储器](#accessors-and-mutators)
    - [定义一个访问器](#defining-an-accessor)
    - [定义一个存储器](#defining-a-mutator)
- [日期调节器](#date-mutators)
- [属性转换](#attribute-casting)
    - [数组 & JSON 转换](#array-and-json-casting)

<a name="introduction"></a>
## 前言

访问器和存储器允许你在获取或者设置 Eloquent 模型属性值时对其进行格式化操作。比如，你可能希望当一个值存储进数据库之前先对其使用 [Laravel encrypter](/docs/{{version}}/encryption) 进行加密操作，并且可以在你通过模型访问的时候自动的进行该属性的解密。

除了可自定义的的访问器和存储器，Eloquent 也可以自动的将日期字段转换为 [Carbon](https://github.com/briannesbitt/Carbon) 实例，或者甚至是将字符串字段转换为 [JSON](#attribute-casting)。

<a name="accessors-and-mutators"></a>
## 访问器 & 存储器

<a name="defining-an-accessor"></a>
### 定义一个访问器

为了定义一个访问器，你需要在你的模型上创建一个类似 `getFooAttribute` 的方法，其中的 `Foo` 是你需要进行访问的列名的驼峰方式的命名。在这个例子中，我们将定义一个 `first_name` 属性的访问器。这个访问器会在 Eloquent 尝试获取 `first_name` 属性值时触发：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Get the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

就如你所看到的，属性原始的值会被传递到访问器中，这允许你对原始值进行操作及返回格式化后的值。你只需要简单的访问 `first_name` 属性就可以从访问器中访问该值：

    $user = App\User::find(1);

    $firstName = $user->first_name;

<a name="defining-a-mutator"></a>
### 定义一个存储器

为了定义一个存储器，你需要在你的模型上定义一个类似 `setFooAttribute` 的方法，其中的 `Foo` 是你期望访问的列的驼峰样式的名称。那么，这一次，让我们为 `first_name` 属性定义一个存储器。这个存储器会在模型尝试设置 `first_name` 属性的值时进行调用：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * Set the user's first name.
         *
         * @param  string  $value
         * @return void
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

存储器会接收即将设置到属性中的值，这允许你对这个值进行操作，并将其设置到模型内部的 `$attributes` 属性中。所以，举个示例，如果我们尝试将 `first_name` 属性设置为 `Sally`:

    $user = App\User::find(1);

    $user->first_name = 'Sally';

在这个例子中，`setFirstNameAttribute` 方法会被调用并伴随 `Sally` 值。存储器会应用 `strtolower` 方法将名字小写化然后将值设置到内部的 `$attributes` 数组中。

<a name="date-mutators"></a>
## 日期调节器

默认的，Eloquent 会转换 `created_at` 和 `updated_at` 列为 [Carbon](https://github.com/briannesbitt/Carbon) 实例，这个实例可以提供多种有用的方法，并且它继承自原生 PHP 的 `DataTime` 类。你可以自定义哪些字段可以进行自动的转换，甚至是完全禁用这种转换，你需要在你的模型中复写 `$dates` 属性：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = [
            'created_at',
            'updated_at',
            'deleted_at'
        ];
    }

当一列被认为是日期时，你可以将其设置为 UNIX 时间戳，日期字符串（`Y-m-d`），时间字符串，和 `DateTime` / `Carbon` 实例，并且日期的值会自动的正确的存储到数据库中：

    $user = App\User::find(1);

    $user->deleted_at = Carbon::now();

    $user->save();

就如上面所述，当获取的属性是 `$dates` 属性所列出的值之一时，它会自动的被转换为 [Carbon](https://github.com/briannesbitt/Carbon) 实例，这允许你在属性上使用 Carbon 的一些方法:

    $user = App\User::find(1);

    return $user->deleted_at->getTimestamp();

#### 日期格式化

默认的，时间戳被格式化为 `Y-m-d H:i:s` 的格式。如果你希望自定义时间戳的格式，你需要在你的模型中设置 `$dateFormat` 属性。该属性将确定日期属性将如何存储到数据库中以及当模型进行序列化或者 JSON 化时如何展示：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

<a name="attribute-casting"></a>
## 属性转换

你可以在你的模型中定义 `$casts` 属性来提供一种方便的方式将属性转换为通用的数据类型。`$casts` 属性应该是一个数组，并且其每一项的键应该是需要进行转换的属性名，而其键所对应的值应该是你需要属性转换到的类型。支持的转换类型有：`integer`，`real`，`float`，`double`，`string`，`boolean`，`object`，`array`，`coolection`，`date`，`datetime`，和 `timestamp`。

比如，让我们转换 `is_admin` 属性，它在数据库中存储的值为一个整型（`0` 或者 `1`），我们将其转换为布尔值：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be casted to native types.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

现在，每当你访问 `is_admin` 属性时，其值都会被转换为布尔值，即使其在数据库中存储的是整型值：

    $user = App\User::find(1);

    if ($user->is_admin) {
        //
    }

<a name="array-and-json-casting"></a>
### 数组 & JSON 转换

`array` 转换的类型对于存储序列化 JSON 值的列尤其有用。比如，如果数据库有一个 `TEXT` 或者 `JSON` 类型的字段，并且其存储的是序列化的 JSON，如果你添加该属性的 `array` 转换，那么当你在 Eloquent 模型上访问这个属性时，它将会自动的进行反序列化为 PHP 的数组：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * The attributes that should be casted to native types.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

当你的转换属性定义完成之后，你可以访问 `options` 属性，并且它会自动的被从 JSON 反序列化为 PHP 数组。当你设置值到 `options` 属性时，所给定的数组会自动的序列化为 JSON 格式，然后进行存储：

    $user = App\User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();
