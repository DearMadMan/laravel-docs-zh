# Laravel Valet

- [前言](#introduction)
    - [Valet 还是 Homestead](#valet-or-homestead)
- [安装](#installation)
- [发行说明](#release-notes)
- [服务站点](#serving-sites)
    - ["Park" 命令](#the-park-command)
    - ["Link" 命令](#the-link-command)
    - [站点安全](#securing-sites)
- [分享站点](#sharing-sites)
- [查看日志](#viewing-logs)
- [自定义 Valet 驱动](#custom-valet-drivers)
- [其它 Valet 命令](#other-valet-commands)

<a name="introduction"></a>
## 前言

Valet 是专为 Mac 平台准备的精简的 Laravel 开发环境。没有 Vagrant，Apache，Nginx。更没有 `/etc/hosts` 文件。你甚至可以使用本地隧道公开分享你的站点。_是的，我们也喜欢它_。

Laravel Valet 总是会在你的机器启动时在后台启动 [Caddy](https://caddyserver.com) 来配置你的 Mac。接着使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)，Valet 会代理所有 `*.dev` 域名的请求指向到你本地机器所安装的站点。

换言之，仅仅只需要使用 7 MB 的内存就可以启动一个速度极快的 Laravel 开发环境。Valte 并不是为了取代 Vagrant 或者 Homestead 而存在的，如果你喜欢灵活的运用知识，更喜欢极限的速速，和更小的内存占用，那么它确实是一种可选的替代方案。

Vlate 可以完美支持以下软件，但是它并不仅仅局限于此：

<div class="content-list" markdown="1">
- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Symfony](https://symfony.com)
- [Zend](http://framework.zend.com)
- [CakePHP 3](http://cakephp.org)
- [WordPress](https://wordpress.org)
- [Bedrock](https://roots.io/bedrock)
- [Craft](https://craftcms.com)
- [Statamic](https://statamic.com)
- [Jigsaw](http://jigsaw.tighten.co)
- Static HTML
</div>

但是不管怎么样，你也可以继承 Valet 创建自己的 [自定义驱动](#custom-valet-drivers)。

<a name="valet-or-homestead"></a>
### Valet 还是 Homestead

就如你所知道的，Laravel 提供了 [Homestead](/docs/{{language}}/{{version}}/homestead)，一个本地的 Laravel 开发环境。Homestead 和 Valet 对于目标受众和对于本地开发的途径是不同的。Homestead 提供了一个完整的 Ubuntu 虚拟机和自动化的 Nginx 配置。而如果你使用的是 Windows / Linux 或者你需要一个完整的虚拟化的 Linux 开发环境，那么 Homestead 将是一个完美的选择。

Valet 只能支持 Mac 平台，并且它需要你在本地主机中安装 PHP 和数据库服务。你可以轻松的通过使用 [Homebrew](http://brew.sh/) 的安装命令来进行本地安装，如 `brew install php70` 和 `brew install mariadb`。Valte 提供了极速的本地开发环境的同时伴随着极低的资源消耗，所以它对于那些仅仅只是使用 PHP / MySQL 并且不需要完整的虚拟化开发环境的开发者来说是及其友好的。

Homestead 和 Valet 对于配置本地开发环境来说都是非常棒的选择，如何选择？这需要你的自我尝试或者配合团队需要来做出决定。

<a name="installation"></a>
## 安装

**Valet 需要 macOS 和 [Homebrew](http://brew.sh/)。你应该确保没有其它的项目如 Apache 或者 Nginx 绑定了本地机器的 80 端口。

<div class="content-list" markdown="1">
- 安装或更新 [Homebrew](http://brew.sh/) 到最新的版本
- 通过使用 Homebrew 的 `brew install homebrew/php/php70` 命令安装 PHP 7.0
- 通过 `composer global require laravel/valet` 命令来安装 Valet。你需要确保 `~/.composer/vendor/bin` 目录已经加入了你的系统 "PATH" 中。
- 执行 `valet install` 命令。这将会进行配置并且安装 Valet 和 DnsMasq，同时在系统中注册在系统启动时启动的 Valet 的守护进程。
</div>

当 Valet 安装完成之后，你应该尝试一下在终端中 ping 任意的 `*.dev` 域名，比如 `ping foobar.dev`。如果 Valet 已经正确的安装了，那么你应该会看到来自 `127.0.0.1` 的响应。

每当你的电脑启动时，Valet 都会自动的启动它的守护进程。所以你不需要再一次的执行 `valet start`或者 `valet install`，因为最初的 Valet 已经是完整的了。

#### 使用其它域名

默认的，Valet 使用 `.dev` 顶级域名来为你的项目提供服务。如果你希望使用其它的域名，那么你可以使用 `valet domain tld-name` 命令。

比如，入股你喜欢使用 `.app` 来代替 `.dev`，那么你执行 `valet domain app` 之后，Valet 就可以自动的为你的项目提供 `*.app` 域名的服务了。

#### 数据

如果你需要使用数据库，你可以尝试一下 MariaDB，你可以执行 `brew install mariadb` 命令来直接安装它。你可以使用 `root` 作为用户名和一个空的字符串作为密码，通过 `127.0.0.1` 连接到数据库中。

<a name="release-notes"></a>
## 发行说明

### Version 1.1.5

1.1.5 版本做出了多种内部改进。

#### 升级指导

在你使用 `composer global update` 更新完成你的 Valet 之后，你需要运行一下 `valet install` 命令。

### Version 1.1.0

1.1.0 发行版带来了多种改进。内置的 PHP 服务器被 [Caddy](https://caddyserver.com/) 取代。引入 Caddy 做出了多种性能上的改进，并且它允许 Valet 站点创造的 HTTP 请求到其它的 Valet 站点时不会像内置的 HTTP 服务器一样带来闭塞。

#### 升级指导

在你使用 `composer global update` 更新完成你的 Valet 之后，你需要运行一下 `valet install` 命令来创建一个新的 Caddy 守护到你的系统中。

<a name="serving-sites"></a>
## 服务站点

当 Valet 安装完成之后，你可以准备启动服务站点了。Valet 提供了两个命令来帮助你服务 Laravel 站点：`park` 和 `link`。

<a name="the-park-command"></a>
**`park` 命令**

<div class="content-list" markdown="1">
- 在你的 Mac 上创建一个新的目录，你可以使用类似 `mkdir ~/Sites` 的命令，然后 `cd ~/Sites` 并且执行 `valet park`。这个命令会在你当前目录进行注册，将其注册为工作目录，这样 Valte 应该搜索这个文件作为工作站。
- 然后，在这个目录中创建一个新的 Laravel 站点: `laravel new blog`。
- 在浏览器中打开 `http://blog.dev`。
</div>

**这就是所有要做的了** 现在，所有在你已经 `parked` 目录中创建的 Laravel 项目都会自动的被提供 HTTP 服务，你可以使用 `http://folder-name.dev` 类似转换后的域名进行访问。

<a name="the-link-command"></a>
**`link` 命令**

`link` 命令也可以被用来为你的 Laravel 站点提供服务。这条命令常用来对单个目录提供单个站点的服务，而不是为整个目录提供服务。

<div class="content-list" markdown="1">
- 使用这个命令，你需要跳转到你的项目目录并且执行 `valet link app-name` 命令。Valet 会自动的创建一个 `~/.valet/Sites` 软连接来指向到你当前的工作目录。
- 然后你就可以通过浏览器访问 `http://app-name.dev` 了。
</div>

你可以使用 `valet links` 命令来查看所有已经连接的目录。你也可以使用 `valet unlink app-name` 来销毁软连接。

<a name="securing-sites"></a>
**TLS 安全站点**

默认的，Valet 通过使用的 HTTP 来服务站点。事实上，如果你喜欢使用 HTTP/2 提供的加密的 TLS 站点服务，你可以使用 `secure` 命令。比如，如果你的站点被服务在 `laravel.dev` 域名商，你可以执行下面的命令来对保护它:

    valet secure laravel

你可以使用 `unsecure` 命令来讲已保护的站点还原为原生的 HTTP 站点。就像 `secure` 命令一样，这个命令接收 host 名称来作为参数：

    valet unsecure laravel

<a name="sharing-sites"></a>
## 分享站点

Valet 甚至包含了外部网络分享站点的命令。而且它并不需要安装额外的软件，这在首次安装 Valet 时就已经被引入了。

为了分享站点，你需要跳转到站点目录并且在终端中执行 `valet share` 命令。这是一个可以公开访问的 URL 已经被复制到你的剪切板里了，你可以直接粘贴到浏览器中来访问它，就是这么简单！

如果想要停止站点的分享，你只需要按住 `Control + C` 键来取消进程就可以了。

<a name="viewing-logs"></a>
## 查看日志

如果你喜欢在终端中查看所有站点的实时日志流，你可以在终端中执行 valet logs 命令。在站点被访问时，终端就会自动的展示最新的日志。这是一种非常好的始终在终端中查看日志的方式。

<a name="custom-valet-drivers"></a>
## 自定义 Valet 驱动

你可以为那些还没有被 Valet 所支持的 框架或者 CMS 编写自己的 Valet 驱动。当你安装 Valet 时，它会自动的生成一个 `~/.valet/Drivers` 目录并且在目录中包含了一个 `SampleValetDriver.php` 文件。这个文件简单的演示了如何编写自定义的驱动。编写一个驱动只需要你实现 3 个方法：`serves`，`isStaticFile`，和 `frontControllerPath`。

所有的 3 个方法都接收 `$sitePath`，`$siteName`，和 `$uri` 值作为它们的参数。`$sitePath` 是你需要提供服务的站点的完整路径。就如 `/Users/Lisa/Sites/my-project`。`$siteName` 是 "host" / "site name"，它只是域名的一部分（`my-project`)。`uri` 是进入的请求 URI （`/foo/bar`)。

当你完成自己的 Valet 驱动之后，你需要将它重命名为 `FrameworkValetDriver.php` 并将它放在 `~/.valet/Drivers` 目录中。比如，如果你为 WordPress 编写了一个 valet 驱动，你的文件名称应该为 `WordPressValetDriver.php`。

让我们来看一下编写自定义 Valet 驱动所应该实现的所有方法的简单实现：

#### `serves` 方法

如果你的驱动应该处理进入的请求，那么 `serves` 方法应该返回 `true`。不然的，这方法应该返回 `false`。所以，在这个方法中你应该尝试判断所给定的 `$sitePath` 所包含的项目类型是不是你所尝试提供服务的。

比如，让我们假装我们在编写 `WordPressValetDriver`。那么我们的 serves 方法应该看起来像这样:

    /**
     * Determine if the driver serves the request.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return void
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

#### `isStaticFile` 方法

`isStaticFile` 方法应该用来判断流入的请求是不是在访问一个静态的文件，比如图片或者 Css 文件。如果文件是静态的，这个方法应该返回静态文件在磁盘中的完整路径。如果流入的请求访问的不是一个静态的文件，那么这个方法应该返回 `false`：

    /**
     * Determine if the incoming request is for a static file.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {note} `isStaticFile` 方法只会在 `serves` 方法返回 `true` 时，并且所请求的 URI 不是 `/` 时才会被调用。

#### `frontControllerPath` 方法

`frontControllerPath` 方法应该返回应用前端控制器的完整路径，通常它是和 `index.php` 等量的：

    /**
     * Get the fully resolved path to the application's front controller.
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="other-valet-commands"></a>
## 其它 Valet 命令

命令  | 描述
------------- | -------------
`valet forget` | 从 "parked" 目录中执行这个命令会从将其从 parked 目录列表中移除该目录
`valet paths` | 查看所有已 "parked" 的路径
`valet restart` | 重启 Valet 守护进程
`valet start` | 开始 Valet 守护进程.
`valet stop` | 停止 Valet 守护进程.
`valet uninstall` | 完整的卸载 Valet 守护进程
