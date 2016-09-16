# 文件系统 / 云存储

- [前言](#introduction)
- [配置](#configuration)
    - [公共磁盘](#the-public-disk)
    - [本地磁盘](#the-local-driver)
    - [驱动需求](#driver-prerequisites)
- [获取磁盘实例](#obtaining-disk-instances)
- [检索文件](#retrieving-files)
    - [文件 URLs](#file-urls)
    - [文件元数据](#file-metadata)
- [存储文件](#storing-files)
    - [文件上传](#file-uploads)
    - [文件可见性](#file-visibility)
- [删除文件](#deleting-files)
- [目录](#directories)
- [自定义文件系统](#custom-filesystems)

<a name="introduction"></a>
## 前言

Laravel 提供了一个强大的文件系统的抽象，这得益于 Frank de Jonge 所开发的 [Flyststem](https://github.com/thephpleague/flysystem) PHP 包。Laravel 的文件系统提供了对一些存储驱动的支持，它们包括本地文件系统，Amazon S3，Rackspace 云存储。更为奇妙的是，它可以通过存储配置选项来切换这些存储系统，这是因为 Laravel 对它们提供了统一的 API 接口。

<a name="configuration"></a>
## 配置

文件系统的配置选项存储在 `config/filesystems.php` 文件中。在这个文件中你可以对所有的磁盘进行配置。每个磁盘选项包含着其所使用的存储驱动及其存储位置。对于 Laravel 默认支持的存储驱动，在这个文件中都有相应的配置示例。所以，你可以简单的在这个文件中修改配置选项就可以使用其强大的功能。

当然，你可以配置多个磁盘，甚至可以使多个磁盘使用相同的驱动。

<a name="the-public-disk"></a>
### 公共磁盘

`public` 磁盘意味着其可以被公开的访问。默认的 `public` 磁盘使用的是 `local` 驱动并且其存储的文件位置是在 `storage/app/public` 目录。如果你想要这个目录下的文件可以在 web 中进行访问，你需要创建一个 `public/storage` 到 `storage/app/public` 的符号链接。这个约定可以使公开访问的文件保持存放在同一个目录中并且可以在使用像 [Envoyer](https://envoyer.io/) 这种无痛持续部署系统时可以方便的共享整个部署过程。

你可以使用 `storage:link` Artisan 命令来快速的创建符号链接：

    php artisan storage:link

当然，一旦文件被存储并且建立了符号链接。你就可以通过 `asset` 帮助方法来生成文件的 URL：

    echo asset('storage/file.txt');

<a name="the-local-driver"></a>
### 本地驱动

当使用 `local` 驱动时，你需要知道的是所有的文件操作都是相对于配置文件中的 `root` 选项所定义的目录。默认的这个值设置的是 `storage/app` 目录。因此，下面的方法将文件存储到 `storage/app/file.txt`:

    Storage::disk('local')->put('file.txt', 'Contents');

<a name="driver-prerequisites"></a>
### 驱动需求

#### Composer 依赖包

在使用 S3 或者 Rackspace 驱动之前，你需要先通过 Composer 来安装适当的包文件：

- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`
- Rackspace: `league/flysystem-rackspace ~1.0`

#### S3 驱动配置

S3 驱动配置相关信息已经存储在了你的 `config/filesystems.php` 配置文件中。这个文件为 S3 驱动提供了一个配置示例。你可以自由的修改这个数组为你的 S3 配置和凭证信息。

#### FTP 驱动配置

Laravel 的文件系统可以很好的支持 FTP 的集成，但是在默认的配置文件中并没有给出示例。如果你需要配置 FTP 的文件系统，你可以使用下面的配置示例：

    'ftp' => [
        'driver'   => 'ftp',
        'host'     => 'ftp.example.com',
        'username' => 'your-username',
        'password' => 'your-password',

        // Optional FTP Settings...
        // 'port'     => 21,
        // 'root'     => '',
        // 'passive'  => true,
        // 'ssl'      => true,
        // 'timeout'  => 30,
    ],

#### Rackspace 驱动配置

Laravel 的文件系统可以很好的支持 Rackspace 的集成，但是在默认的配置文件中并没有给出示例。如果你需要配置 Rackspace 文件系统，你可以使用下面的示例：

    'rackspace' => [
        'driver'    => 'rackspace',
        'username'  => 'your-username',
        'key'       => 'your-key',
        'container' => 'your-container',
        'endpoint'  => 'https://identity.api.rackspacecloud.com/v2.0/',
        'region'    => 'IAD',
        'url_type'  => 'publicURL',
    ],

<a name="obtaining-disk-instances"></a>
## 获取磁盘实例

`Storage` 假面可以用来和你所配置的磁盘进行交互。比如，你可以使用它的 `put` 方法来将用户的头像图片存储到默认的磁盘。如果你在调用该方法的时候没有在其之前使用 `disk` 方法的话，那么该方法会自动的将头像传递到默认的磁盘中：

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $fileContents);

当你使用多个磁盘时，你可以使用 `Storage` 假面的 `disk` 方法来指定所需访问的磁盘。当然，你可以使用链式的方法来进行持续的操作：

    Storage::disk('s3')->put('avatars/1', $fileContents);

<a name="retrieving-files"></a>
## 检索文件

`get` 方法可以用来从所给定的文件中检索出其内容。该方法会返回文件的原始字符串内容，你需要注意的是，所有的文件路径应该被指定为相对于磁盘配置中的 "root" 路径：

    $contents = Storage::get('file.jpg');

`exists` 方法可以用来判断所给定的文件是否存在于指定的磁盘中：

    $exists = Storage::disk('s3')->exists('file.jpg');

<a name="file-urls"></a>
### 文件 URLs

当使用 `local` 或者 `s3` 驱动时，你可以使用 `url` 方法来获得给定文件的 URL。如果你使用的是 `local` 驱动，那它只会简单的在给定的路径前增加 `/storage` 前缀以返回相对的文件路径。如果你使用的是 `s3` 驱动，将会返回完整的远端 URL：

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file1.jpg');

> {note} 当使用的是 `local` 驱动时，所有的文件应该被存放在 `storage/app/public` 目录中，并且它应该是可以公开访问的。另外，你应该 [创建一个符号链接](#the-public-disk) 到 `storage/app/public` 目录。

<a name="file-metadata"></a>
### 文件元数据

除了可以进行读写文件的操作，Laravel 也可以提供文件的相关信息。比如，你可以使用 `size` 方法来获取文件的字节大小：

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file1.jpg');

`lastModified` 方法会返回给定文件的最后修改时间，它使用的是 UNIX 时间戳：

    $time = Storage::lastModified('file1.jpg');

<a name="storing-files"></a>
## 存储文件

`put` 方法可以用来将文件存储到磁盘。你可以传递一个 PHP 的 `resource` 到 `put` 方法，那么它会使用文件系统的底层流支持。在与大型文件交互时，是非常推荐使用文件流的：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### 自动流

如果你希望 Laravel 可以自动的以流文件的方式传递到你的存储中，那么你可以使用 `putFile` 或者 `putFileAs` 方法。这些方法接收一个 `Illuminate\Http\File` 或者 `Illuminate\Http\UploadedFile` 实例，并且会自动的将流文件传递到你所希望的存储位置：

    use Illuminate\Http\File;

    // Automatically calculate MD5 hash for file name...
    Storage::putFile('photos', new File('/path/to/photo'));

    // Manually specify a file name...
    Storage::putFile('photos', new File('/path/to/photo'), 'photo.jpg');

关于 `putFile` 方法，你需要了解一些重要的事情。你需要注意我们只指定了目录名称，而没有文件名称。默认的，`putFile` 方法会自动的依据文件的内容来追踪计算 MD5 hash，并以这个值来作为文件的名称。同时 `putFile` 方法会返回该文件的路径，路径中包含了文件的名称，所以你可以将路径存储到你的数据库中。

`putFile` 和 `putFileAs` 方法也接收一个参数用来指定所存储文件的可见性。这常用于在云存储如 S3 中存储文件的同时并将为其开放公开访问的权限：

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

#### 前置 & 追加内容到文件

`prepend` 和 `append` 方法允许你轻松的往文件的起始或结束位置加入内容：

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

#### 复制 & 移动文件

`copy` 方法可以用来复制磁盘中已存在的文件到新的位置，而 `move` 方法可以被用对磁盘中已存在的文件进行名称的修改或移动到新的位置：

    Storage::copy('old/file1.jpg', 'new/file1.jpg');

    Storage::move('old/file1.jpg', 'new/file1.jpg');

<a name="file-uploads"></a>
### 文件上传

在 web 应用中，其中一个常用的场景就是存储用户上传的文件到系统中，比如用户的头像，照片和文档。Laravel 可以非常简单的使用已上传文件的 `store` 方法将其进行存储。你只需要简单的在 `store` 方法中传递你所需要存储到的文件路径就可以了：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserAvatarController extends Controller
    {
        /**
         * Update the avatar for the user.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

在上面的示例中，你需要注意一些事情。你应该注意到我们在存储时只指定了目录名称，而没有指定文件名称。默认的，`store` 方法会自动的基于文件的内容来生成文件的名称。这会自动的依据文件的内容来追踪技术 MD5 hash。同时 `store` 方法会返回文件的路径，包括所生成的文件名称，这样你就可以将其存储到你的数据库中了。

你也可以像上面一样调用 `Storage` 假面的 `putFile` 方法来提供相同的文件操作：

    $path = Storage::putFile('avatars', $request->file('avatar'));

> {note} 如果你接收非常大的上传文件，你也可以参考下面的示例手动的指定文件的名称。为极大的文件计算 MD5 hash 可能会造成内存紧张。

#### 指定文件名称

如果你不喜欢所存储的文件被自动的分配名称。那么你可以使用 `storeAs` 方法，它可以接收路径，文件的名称和一个可选的磁盘作为参数：

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

当然，你也可以在 `Storage` 假面上调用 `putFileAs` 方法，这将像上面一样提供同样的文件操作：

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

#### 指定磁盘

默认的，这个方法会使用默认的磁盘，如果你希望指定使用其它的磁盘，你可以传递磁盘的名称作为第二个参数到 `store` 方法：

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

<a name="file-visibility"></a>
### 文件可见性

在 Laravel 的文件系统集成中，可见性是跨多个平台文件权限的一种抽象。文件可以被指定为 `public` 或者 `private`。当文件被指定为 `public` 时，这表明通常这个文件应该是可以被它人访问的。比如，当使用 S3 驱动时，你可以从 `public` 文件中取回 URLs。

你可以通过使用 `put` 方法来设置文件的可见性：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

如果文件已经被存储了，那么它的可见性可以被 `getVisibility` 和 `setVisibility` 方法检索和设置：

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public')

<a name="deleting-files"></a>
## 删除文件

`delete` 方法可以接受一个文件名或者文件名所组成的数组，它将从磁盘中删除相应的文件：

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file1.jpg', 'file2.jpg']);

<a name="directories"></a>
## 目录

#### 从目录中获取所有文件

`files` 方法会返回所给定目录中所有的文件所组成的数组。如果你想要在检索到的文件中包含所给定目录的子目录。那么你需要使用 `allFiles` 方法：

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### 从目录中获取所有目录

`directories` 方法可以返回所给定目录下的所有子目录所组成的数组。另外你可以使用 `allDirectories` 方法递归检索子目录：

    $directories = Storage::directories($directory);

    // Recursive...
    $directories = Storage::allDirectories($directory);

#### 创建一个目录

`makeDirectory` 方法将创建给定的目录，包括其所需要的子目录：

    Storage::makeDirectory($directory);

#### 删除一个目录

最后，`deleteDirectory` 方法可以用来删除一个目录，并且其删除磁盘中该目录下所有的文件：

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## 自定义文件系统

Laravel 的文件系统对几种常见的存储系统提供了开箱即用的支持。事实上，文件系统并没有限制你只使用所提供的这些。你可以自己创建一个适配器来构建一个自定义的驱动去支持你所期望使用的文件存储系统。

你需要创建一个 [服务提供者](/{{language}}/{{version}}/providers) 来进行自定义文件存储系统的构建。比如 `DropboxServiceProvider`。在提供者的 `boot` 方法中，你需要使用 `Storage` 假面的 `extend` 方法来定义你自己的驱动：

    <?php

    namespace App\Providers;

    use Storage;
    use League\Flysystem\Filesystem;
    use Dropbox\Client as DropboxClient;
    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Dropbox\DropboxAdapter;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function($app, $config) {
                $client = new DropboxClient(
                    $config['accessToken'], $config['clientIdentifier']
                );

                return new Filesystem(new DropboxAdapter($client));
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

`extend` 方法中的第一个参数应该是驱动的名称，第二个参数是一个闭包，闭包接受 `$app` 和 `$config` 变量。被解析的闭包必须返回一个 `League\Flysystem\Filesystem` 的实例。`$config` 变量包含了 `config/filesystems.php` 文件中指定磁盘的值。

一旦你创建了服务提供者并且注册了这个扩展，你就可以在 `config/filesystem.php` 配置文件中使用 `dropbox` 驱动了。
