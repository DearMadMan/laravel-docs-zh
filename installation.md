# 安装

- [安装](#installation)
    - [服务端依赖](#server-requirements)
    - [安装 Laravel](#installing-laravel)
    - [配置](#configuration)

<a name="installation"></a>
## 安装

<a name="server-requirements"></a>
### 服务端依赖

Laravel 框架依赖于一些小型系统。当然，这些需求依赖在 [Laravel Homestead](/docs/{{language}}/{{version}}/homestead) 都有完美的提供。所以，我们非常推荐你使用 Homestead 来作为本地的 Laravel 开发环境。

不管怎么样，如果你不喜欢使用 Homestead，那么你需要确保你的服务端安装有以下依赖:

<div class="content-list" markdown="1">
- PHP >= 5.6.4
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
</div>

<a name="installing-laravel"></a>
### 安装 Laravel

Laravel 利用 [Composer](http://getcomposer.org) 来管理它的依赖，所以，在你使用 Laravel 之前，请确保你在你的机器中已经安装了 Composer。
Laravel utilizes  to manage its dependencies. So, before using Laravel, make sure you have Composer installed on your machine.

#### 通过 Laravel 提供的安装程序

首先，你需要使用 Composer 来下载 Laravel 所提供的安装程序:

    composer global require "laravel/installer"

请确保目录 `~/.composer/vendor/bin` (或者基于你系统当量的目录) 已被添加到 PATH 中，只有这样 `laravel` 命令才能被你的系统所定位。

一旦你安装完成，你就可以使用 `laravel new` 命令来在你指定的目录中生成一个新的 Laravel 应用。比如，我们可以使用 `laravel new blog` 命令来创建被命名为 `blog` 的目录，该目录中被放置了一个新的 Laravel 应用，并且应用中所需依赖也全部安装完毕。这种安装方式会比通过 Composer 安装要快速的多：

    laravel new blog

#### 通过 Composer Create-Project 命令

另外，你可以在你的终端中使用 Composer 的 `create-project` 命令来安装 Laravel：

    composer create-project --prefer-dist laravel/laravel blog

<a name="configuration"></a>
### 配置

#### 公共目录

在你安装完成之后，你应该设置你应用的 web 根目录为 `public` 目录。这个目录中的 `index.php` 是作为所有进入应用的 HTTP 请求前端控制器来进行服务的。

#### 配置文件

所有的 Laravel 框架的配置文件都被存贮在 `config` 目录中。所有的配置选项都被加以文档标注，所以你可以自由的浏览这些配置文件，利用这些标注来熟悉配置选项。

#### 目录权限

在你安装 Laravel 之后，你也需要配置一些权限问题。目录中的 `storage` 和 `bootstrap/cache` 应该对与你的 web 服务具有可写的权限，不然 Laravel 没法正常运行。如果你使用 [Homestead](/docs/{{language}}/{{version}}/homestead) 虚拟机，那么这些权限问题应该都已经被正确设置好了。

#### 应用秘钥

在安装之后的下一件事就应该是设置应用的密钥为一个随机的字符串。如果你是通过 Composer 或者 Laravel 提供的安装程序进行的安装，那么这个密钥应该已经通过 `php artisan key:generate` 命令正确设置好了。通常的，这个字符串应该是一个 32 个字符长度。这个密钥可以被设置在 `.env` 环境文件中。如果你还没有将 `.env.example` 文件重命名为 `.env`，那么你现在就应该做了。**如果你的应用密钥没有被设置，那么你的 session 和其他加密的数据都将不是安全的！**

#### 额外的配置

Laravel 附带的其他功能几乎不需要额外的配置了。你已经可以自用的进行开发了。但是无论如何，你不妨回顾一下 `config/app.php` 文件和它的文档。它包含了一些如 `timezone` 和 `locale` 这些你希望改变的应用选项。

你也可能希望配置一些额外的 Laravel 组件，比如：

<div class="content-list" markdown="1">
- [Cache](/docs/{{language}}/{{version}}/cache#configuration)
- [Database](/docs/{{language}}/{{version}}/database#configuration)
- [Session](/docs/{{language}}/{{version}}/session#configuration)
</div>

当你的 Laravel 程序安装完毕，你应该也 [配置一下你的本地开发环境](/docs/{{language}}/{{version}}/configuration#environment-configuration).
