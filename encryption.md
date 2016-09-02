# 加密

- [前言](#introduction)
- [配置](#configuration)
- [使用加密器](#using-the-encrypter)

<a name="introduction"></a>
## 前言

Laravel 的加密器是使用的 OpenSSL 来提供 AES-256 和 AES-128 加密。我们极力推荐你使用 Laravel 內建的加密器而不是你自己构建的加密算法。Laravel 中所有加密值的签证都是使用的消息鉴别码（MAC）算法，所以加密后的消息值，它们是无法修改的。

<a name="configuration"></a>
## 配置

在你使用 Laravel 的加密之前，你应该先在你的 `config/app.php` 文件中设置 `key` 选项。你应该使用 `php artisan key:generate` 命令来生成这个 key 。如果这个值没有设置正确，那么 Laravel 中所有的加密都是不安全的。

<a name="using-the-encrypter"></a>
## 使用加密器

#### 加密一个值

你可以使用 `encrypt` 帮助方法来加密一个值。所有的加密都是使用的 OpenSSL 和 `AES-256-CBC` 算法。除此之外，所有加密后的值都会伴随一个消息认证码（MAC）用来检测任何的消息变动。

    <?php

    namespace App\Http\Controllers;

    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => encrypt($request->secret)
            ])->save();
        }
    }

> {note} 加密后的值在加密期间是通过 `serialize` 来传递的。这意味着你可以对对象和数组进行加密。所以，非 PHP 客户端在接收加密后的值之后还需要对数据进行反序列化操作。

#### 解密一个值

你可以使用 `decrypt` 帮助方法来对值进行解密。如果值不能被解密，比如 MAC 验证失败，那么会抛出一个 `Illuminate\Contracts\Encryption\DecryptException` 异常：

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
