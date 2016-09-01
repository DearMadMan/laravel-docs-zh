# 哈希

- [前言](#introduction)
- [基础用法](#basic-usage)

<a name="introduction"></a>
## 前言

Laravel 的 `Hash` [假面](/docs/{{language}}/{{version}}/facades) 对存储用户的密码提供了安全的 Bcrypt 加密工具哈希化。如果你使用了 Laravel 自带的 `LoginController` 和 `RegisterController`  控制器，那么你就已经使用过了它。它会在注册和认证时自动的使用 Bcrypt 加密工具。

> {tip} Bcrypt 对于密码哈希化是一种很好的选择，因为它的‘工作因子’是可以调整的，这就意味着随着硬件功率的提升，生成哈希密码的时间也会提升。

<a name="basic-usage"></a>
## 基础用法

你可以使用 `Hash` 假面的 `make` 方法来进行密码的哈希化：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use App\Http\Controllers\Controller;

    class UpdatePasswordController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // Validate the new password length...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

#### 针对 Hash 密码的验证

`check` 方法允许你验证所给定的哈希化的密码是否与所给定的原始字符串相匹配。如果你使用了 [Laravel](/docs/{{language}}/{{version}}/authentication) 所提供的 `LoginController` 控制器，那么你并不需要直接的去使用这个方法，因为在这个控制器中系统会自动的调用这个方法：

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

#### 检查密码是否需要刷新


`needsRehash` 方法可以用来判断密码是否需要重新生成，当工作因子变更时而密码没有使用新的工作因子生成时，其将返回 `true`:

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
