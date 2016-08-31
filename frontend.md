# JavaScript & CSS

- [前言](#introduction)
- [编写 CSS](#writing-css)
- [编写 JavaScript](#writing-javascript)
    - [编写 Vue 组件](#writing-vue-components)

<a name="introduction"></a>
## 前言

虽然 Laravel 并没有明确限定你使用哪种 JavaScript 或者 CSS 预处理器，但是它通过使用 [Bootstrap](http://getbootstrap.com) 和 [Vue](https://vuejs.org) 来提供基础的起点。这对于很多应用来说是非常有用的。默认的，Laravel 使用 [NPM](https://npmjs.org) 来安装上述框架及所有的前端包。

#### CSS

[Laravel Elixir](/docs/{{language}}/{{version}}/elixir) 提供了一种简洁丰富的 API 来指导 SASS 或者 Less 的编译，SASS 或者 Less 是原生 CSS 的扩展，它们为 CSS 添加了变量，混合和其它的一些强大的特性，这使你能够更轻松愉悦的编写和管理你的 CSS。

在这个文档中，我们将会短暂的来探讨一下大体的 CSS 编译过程。事实上，你应该查阅完整的 [Laravel Elixir 文档](/docs/{{language}}/{{version}}/elixir) 来了解更多的 SASS 或 Less 编译相关的信息。

#### JavaScript

Laravel 并没有限制你使用指定的框架或类库来构建你的应用。事实上，你也可以一点都不使用 JavaScript。但是 Laravel 确实是引入了一些基础的脚手架来加速应用的开发，我们使用 [Vue](https://vuejs.org) 类库来编写现代的 JavaScript。Vue 提供了一些丰富的 API 来通过使用组件化的方式构建健壮的 JavaScript 应用。

<a name="writing-css"></a>
## 编写 CSS

Laravel 的 `package.json` 文件中引入了 `bootstrap-sass` 包来帮助你使用 Bootstrap 快速的构建应用的原型。然而，你可以自由的在 `package.json` 文件中删除或者添加包到你的应用中。你也没有必要去使用 Bootstrap 框架来构建你的应用 - 它只是为那些需要的人提供的一个极好的起点。

在编译你的 CSS 之前，你需要先通过 NPM 安装项目中前端所需的依赖：

    npm install

当执行 `npm install` 安装完依赖之后，你可以使用 [Gulp](http://gulpjs.com/) 来编译你的 SASS 文件到原生的 CSS。`gulp` 命令会依据你的 `gulpfile.js` 文件的指导来进行编译处理。通常，你所编译完成的 CSS 文件会被存放在 `public/css` 目录下:

    gulp

Laravel 默认包含的 `gulpfile.js` 将会编译 `resources/assets/sass/app.scss` SASS 文件。这个 `app.scss` 文件只是简单的引入 Bootstrap 中的 SASS 变量，这对于大多数应用来说是提供了一个良好的开始。你可以自由的根据喜好来修改 `app.scss` 文件，或者你也可以通过 [配置 Laravel Elixir](/docs/{{language}}/{{version}}/elixir) 来使用完全不同的预处理器。

<a name="writing-javascript"></a>
## 编写 JavaScript

应用中所有的 JavaScript 依赖都可以在项目根目录下的 `package.json` 文件中找到。这个文件和 `composer.json` 文件很像，只不过它使用 JavaScript 依赖替换了 PHP 依赖。你可以通过使用 [Node 包管理 (NPM)](https://npmjs.org) 来安装这些依赖:

    npm install

默认的，Laravel 的 `package.json` 文件中引入了一些如 `vue` 和 `vue-resource` 之类的包来帮助你更快的构建 JavaScript 应用。你可以根据自己的需要自由的添加或者删除 `package.json` 文件中的依赖项。

当包安装完毕之后，你可以使用 `gulp` 命令来 [编译你的资源](/docs/{{language}}/{{version}}/elixir)。Gulp 是为 JavaScript 提供的一种命令行构建系统。当你执行 `gulp` 命令时，Gulp 会依据 `gulpfile.js` 文件的指导来执行:

    gulp

默认的，Laravel 的 `gulpfile.js` 文件编译你的 SASS 文件和 `resources/assets/js/app.js` 文件。在 `app.js` 文件中，你可以注册你的 Vue 组件，或者，如果你更喜欢一些其它的前端框架，你可以配置你自己的 JavaScript 应用。通常你所编译完成的 JavaScript 文件将被存储在 `public/js` 目录中。

> {tip} `app.js` 文件会加载 `resources/assets/js/bootstrap.js` 文件，而 `bootstrap.js` 会启动和配置 Vue，Vue Resource，jQuery，和一些其它的 JavaScript 依赖。如果你还有一些其它的 JavaScript 依赖需要配置，你也可以在这个文件中进行配置。

<a name="writing-vue-components"></a>
### 编写 Vue 组件

默认的，一个新安装的 Laravel 应用包含了一个 `Example.vue` Vue 组件。它被存放在 `resources/assets/js/components` 目录。`Example.vue` 文件是一个 [单文件 Vue 组件](https://vuejs.org/guide/application.html#Single-File-Components) 的示例，它在一个文件中同时定义 JavaScript 和 HTML 模板。单文件组件提供了一种方便的途径来构建 JavaScript 应用。这个示例组件被注册在 `app.js` 文件中:

    Vue.component('example', require('./components/Example.vue'));

为了在你的应用中使用组件，你可以简单的将其放置在你的 HTML 模板中。比如，在执行 `make:auth` Artisan 命令之后，Laravel 会为你应用的认证生成一些脚手架并注册一些视图，你可以将组件放置在 `home.blade.php` Blade 模板中:

    @extends('layouts.app')

    @section('content')
        <example></example>
    @endsection

> {tip} 你需要注意的是，你应该在每次修改 Vue 组件之后都执行 `gulp` 命令，或者，你可以执行 `gulp watch` 命令来对文件进行监控，它们会在每次被修改时自动的进行重新编译。

当然，如果你对 Vue 组件的编写很感兴趣，那么你应该阅读 [Vue 文档](http://vuejs.org/guide/)，它为整个 Vue 框架提供了彻底易读的概述。
