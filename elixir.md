# 编译资源 (Laravel Elixir)

- [前言](#introduction)
- [安装 & 设置](#installation)
- [运行 Elixir](#running-elixir)
- [与样式交互](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [原生 CSS](#plain-css)
    - [Source Maps](#css-source-maps)
- [与脚本交互](#working-with-scripts)
    - [Webpack](#webpack)
    - [Rollup](#rollup)
    - [Scripts](#javascript)
- [复制文件 & 目录](#copying-files-and-directories)
- [版本化 / 缓存爆破](#versioning-and-cache-busting)
- [BrowserSync](#browser-sync)

<a name="introduction"></a>
## 前言 

Laravel Elixir（炼金药） 为你的应用定义基础的 [Gulp](http://gulpjs.com/) 任务提供了简单流利的 API。Elixir 提供了几种常用的 CSS 和 JavaScript 预处理器，比如 [Sass](http://sass-lang.com) 和 [Webpack](https://webpack.github.io/)。Elixir 允许你通过链式调用来对你的资源管道进行流利的操作。比如：

```javascript
elixir(function(mix) {
    mix.sass('app.scss')
       .webpack('app.js');
});
```

如果你曾对如何使用 Gulp 和资源预编译有所疑惑，那么你肯定会爱上 Laravel Elixir。事实上，你也可以在开发应用的时候不使用它。你可以自由的使用任何的资源管道工具，或者一点也不用。

<a name="installation"></a>
## 安装 & 设置

#### 安装 Node

在触碰到 Elixir 之前，你首先需要确定你的机器中已经安装了 Node.js 和 NPM:

    node -v
    npm -v

默认的，Laravel Homestead 包含了所有你所需要的；事实上，如果你并不是使用的 Vagrant，你也是可以非常简单的通过 [这里](http://nodejs.org/download/) 来进行安装 Node 和 NPM。

#### Gulp

接着，你需要通过 NPM 中安装 [Gulp](http://gulpjs.com/) 到全局:

    npm install --global gulp-cli

#### Laravel Elixir

最后剩下的就是安装 Elixir 了！在一个新的 Laravel 应用中，你会在根目录中发现 `package.json` 文件。默认的 `package.json` 文件中被引入了 Elixir 和 Webpack JavaScript 模块打包器。你可以把它想象成 `composer.json` 文件。它的不同之处就是它定义的是 Node 依赖，而不是 PHP。你可以通过下面的命令来安装这些依赖：

    npm install

如果你是在 Windows 系统中进行开发，或者运行在虚拟机中的 Windows 系统，你需要运行 `npm install` 命令的同时添加 `--no-bin-links` 选项：

    npm install --no-bin-links

<a name="running-elixir"></a>
## 运行 Elixir

Elixir 是建立于 [Gulp](http://gulpjs.com) 之上的，所以运行 Elixir 任务你只需要在终端运行 `gulp` 命令就行了。添加 `--production` 标识到命令会指导 Elixir 去压缩你的 CSS 和 JavaScript 文件：

    // Run all tasks...
    gulp

    // Run all tasks and minify all CSS and JavaScript...
    gulp --production

在上面的命令执行时，你会在相关任务执行时看到友好的事件摘要信息。

#### 监控资源文件的变动

为了不让你每次文件变动之后还要重新运行 `gulp` 命令，你应该使用 `gulp watch` 命令来监控资源文件的变动。这个命令会持续的在你的终端中运行。当检测到资源文件的变动，新的文件将自动编译完成：

    gulp watch

<a name="working-with-stylesheets"></a>
## 与样式文件交互

在你项目的根目录中有一个 `gulpfile.js` 文件，该文件包含了所有的 Elixir 任务。Elixir 任务可以被链式的调用，会通过有序的传递来对你的资源文件进行编译操作。

<a name="less"></a>
### Less

你可以使用 `less` 方法来将 [Less](http://lesscss.org/) 文件编译为 CSS。`less` 方法假设你的 less 文件存放在 `resources/assets/less` 目录中。默认的，在下面的示例中，任务运行的结果将编译的 CSS 文件存放在 `public/css/app.css` 文件中:

```javascript
elixir(function(mix) {
    mix.less('app.less');
});
```

你也可以合并多个 Less 文件到一个单独的 CSS 文件中。默认的，他们将被编译为 `public/css/app.css` 文件：

```javascript
elixir(function(mix) {
    mix.less([
        'app.less',
        'controllers.less'
    ]);
});
```

如果你希望将编译的文件存放到自定义的位置，那么你可以传递第二个参数到 `less` 方法：

```javascript
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets');
});

// Specifying a specific output filename...
elixir(function(mix) {
    mix.less('app.less', 'public/stylesheets/style.css');
});
```

<a name="sass"></a>
### Sass

`sass` 方法允许你将 [Sass](http://sass-lang.com/) 文件编译为 CSS。它假定你的 Sass 文件存放在 `resources/assets/sass`，你可以像下面的方式来使用该方法：

```javascript
elixir(function(mix) {
    mix.sass('app.scss');
});
```

就像 `less` 方法一样，你可以编译多个 Sass 文件到一个 CSS 文件中，还可以将编译的结果存放到指定的位置：

```javascript
elixir(function(mix) {
    mix.sass([
        'app.scss',
        'controllers.scss'
    ], 'public/assets/css');
});
```

#### 自定义路径

虽然非常推荐你使用 Laravel 默认的资源目录，但是如果你确实需要使用不同的目录作为路径的话，你可以以 `./` 为起点作为任意文件的路径。这会指导 Elixir 将文件初始路径导向项目的根目录，而不是使用默认的基础目录。

比如，为了将放置在 `app/assets/sass/app.scss` 的文件编译到 `public/css/app.css`，你可以这么调用 `sass` 方法:

```javascript
elixir(function(mix) {
    mix.sass('./app/assets/sass/app.scss');
});
```

<a name="stylus"></a>
### Stylus

`stylus` 方法可以用来编译 [Stylus](http://stylus-lang.com/) 到 CSS。它假定你的 Stylus 文件被存储在 `resources/assets/stylus` 目录，你可以像下面这样的调用这个方法：

```javascript
elixir(function(mix) {
    mix.stylus('app.styl');
});
```

> {tip} 该方法的用法和 `mix.less()` 与 `mix.sass()` 一致。

<a name="plain-css"></a>
### 原生 CSS

如果你希望合并多个 CSS 文件到一个文件中，你可以使用 `styles` 方法。你需要传递文件的路径是相对于 `resources/assets/css` 目录的，并且默认的合并的结果将会被存放到 `public/css/all.css`:

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ]);
});
```

当然，你也是可以自定义输出结果的路径：

```javascript
elixir(function(mix) {
    mix.styles([
        'normalize.css',
        'main.css'
    ], 'public/assets/css/site.css');
});
```

<a name="css-source-maps"></a>
### Source Maps

在 Elixir 中，编译地图默认是开启的。这个地图文件可以使你在浏览器中追踪到编译前代码的位置，这样方便于调试。对于所有的被编译后的文件你都可以在相同的目录下发现 `*.css.map` 文件。

如果你不希望生成地图，你可以使用 `sourcemaps` 选项进行关闭：

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
    mix.sass('app.scss');
});
```

<a name="working-with-scripts"></a>
## 与脚本交互

Elixr 也提供了多种方法来帮助你协同 JavaScript 的工作，例如 ECMAPScript 2015 的编译，模块的打包，脚本的压缩，或者是简单的原生 JavaScript 文件的合并，这都不是问题！

当编写 ES2015 与模块时，你可以选择使用 [Webpack](http://webpack.github.io) 和 [Rollup](http://rollupjs.org/)。如果你对这些都很陌生，别担心，Elixir 会在背后哦处理所有困难的工作。默认的 Laravel `gulpfile` 使用 `webpack` 来编译 JavaScript，但是你可以选择你喜欢的模块打包器来使用。

<a name="webpack"></a>
### Webpack

`webpack` 方法可以用来编译和打包 [ECMAScript 2015](https://babeljs.io/docs/learn-es2015/) 到原生的 JavaScript。该方法接收一个相对于 `resources/assets/js` 目录的路径作为参数并生成一个打包后的文件放置在 `public/js` 目录:

```javascript
elixir(function(mix) {
    mix.webpack('app.js');
});
```

如果你需要改变默认的输出路径，你可以简单的指定你所期望的路径，你需要前置 `.`，然后指定相对于项目根目录的路径。比如，编译 `app/assets/js/app.js` 到 `public/dist/app.js`：

```javascript
elixir(function(mix) {
    mix.webpack(
        './resources/assets/js/app.js',
        './public/dist'
    );
});
```

如果你希望使用更多的 Webpack 的功能，那么你可以在根目录下 [配置](https://webpack.github.io/docs/configuration.html) `webpack.config.js`，Elixir 将会读取它的配置进行构建处理。


<a name="rollup"></a>
### Rollup

和 Webpack 相似，Rollup 为 ES2015 提供的新一代的打包器。`rollup` 方法接收一个相对于 `resources/assetjs/js` 路径的文件数组，并且生成一个单独的文件存放在 `public/js` 目录：

```javascript
elixir(function(mix) {
    mix.rollup('app.js');
});
```

就像 `webpack` 方法，你在 `rollup` 方法中也可以自定义输入的路径和输出的路径:

    elixir(function(mix) {
        mix.rollup(
            './resources/assets/js/app.js',
            './public/dist'
        );
    });

<a name="javascript"></a>
### Scripts

如果你想要将多个 JavaScript 文件合并到一个文件中，你可以使用 `scripts` 方法，它会自动的提供编译地图，合并和压缩功能。

`scripts` 方法假设你所有的文件都相对于 `resouces/assets/js` 目录，并且会默认的将结果编译到 `public/js/all.js` 文件中：

```javascript
elixir(function(mix) {
    mix.scripts([
        'order.js',
        'forum.js'
    ]);
});
```

如果你需要合并多个文件到多个不同的路径，你可以通过多次链式调用 `scripts` 方法并传递第二个参数作为指定输出的路径：

```javascript
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

如果你需要合并指定目录下的所有脚本文件，你可以使用 `scriptIn` 方法。合并的结果将存放到 `public/js/all.js`:

```javascript
elixir(function(mix) {
    mix.scriptsIn('public/js/some/directory');
});
```

> {tip} 如果打算去连接多个已经压缩后的供应商类库，比如 jQuery，那么你可以考虑使用 `mix.combine()`。这会在合并这些文件的同时省去编译地图和压缩的步骤。这样可以大大的提供编译的时间。


<a name="copying-files-and-directories"></a>
## 复制文件 & 目录

`copy` 方法可以用来复制文件和目录到一个新的位置。所有的操作都是相对于项目的根目录：

```javascript
elixir(function(mix) {
	mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});
```

<a name="versioning-and-cache-busting"></a>
## 版本化 / 缓存爆破

对于许多开发者比较痛苦的事就是手动的对资源文件增加时间戳或者唯一的 token 标识来强迫浏览器重新加载新的资源文件。Elixir 可以通过 `version` 方法来帮你自动的完成这些。

`version` 方法接收文件名称相对于 `public` 目录，并且它会自动的为文件名增加一个独特的 hash，这样就可以自动的进行缓存清除了。比如，新生成的文件名看上去像这样：`all-16d570a7.css`:

```javascript
elixir(function(mix) {
    mix.version('css/all.css');
});
```

在生成版本化的文件之后，你可以使用 Laravel 的全局帮助函数 `elixir` 在你的 [视图](/{{language}}/{{version}}/views) 文件中进行加载适当的 hashed 资源。`elixir` 方法会自动的判断文件的名称：

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

#### 对多个文件进行版本化


你可以传递一个数组到 `version` 方法来进行多个文件的版本化：

```javascript
elixir(function(mix) {
    mix.version(['css/all.css', 'js/app.js']);
});
```

一旦文件本版本化，你就可以使用 Laravel 的 `elixir` 方法去生成版本化的 links。记住，你只需要向 `elixir` 帮助方法中传递文件名的前缀就可以了，并不需要填写 hash 后的文件名。帮助方法会自动的识别 hash 后的文件名：

    <link rel="stylesheet" href="{{ elixir('css/all.css') }}">

    <script src="{{ elixir('js/app.js') }}"></script>

<a name="browser-sync"></a>
## BrowserSync

BrowserSync 可以在你的前端资源文件变更之后自动的刷新的你浏览器。`browserSync` 方法接收一个 JavaScript 对象并伴随 `proxy` 属性包含应用的本地 URL。然后，当你执行 `gulp watch` 命令之后，你可以通过 300 端口（`http://project.dev:3000`）来访问你的 web 应用并享受浏览器同步的便利:

```javascript
elixir(function(mix) {
    mix.browserSync({
        proxy: 'project.dev'
    });
});
```
