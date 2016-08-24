# JavaScript & CSS

- [Introduction](#introduction)
- [Writing CSS](#writing-css)
- [Writing JavaScript](#writing-javascript)
    - [Writing Vue Components](#writing-vue-components)

<a name="introduction"></a>
## Introduction

While Laravel does not dictate which JavaScript or CSS pre-processors you use, it does provide a basic starting point using [Bootstrap](http://getbootstrap.com) and [Vue](https://vuejs.org) that will be helpful for many applications. By default, Laravel uses [NPM](https://npmjs.org) to install both of these frontend packages.

#### CSS

[Laravel Elixir](docs/{{language}}/{{version}}/elixir) provides a clean, expressive API over compiling SASS or Less, which are extensions of plain CSS that add variables, mixins, and other powerful features that make working with CSS much more enjoyable.

In this document, we will briefly discuss CSS compilation in general; however, you should consult the full [Laravel Elixir documentation](docs/{{language}}/{{version}}/elixir) for more information on compiling SASS or Less.

#### JavaScript

Laravel does not require you to use a specific JavaScript framework or library to build your applications. In fact, you don't have to use JavaScript at all. However, Laravel does include some basic scaffolding to make it easier to get started writing modern JavaScript using the [Vue](https://vuejs.org) library. Vue provides an expressive API for building robust JavaScript applications using components.

<a name="writing-css"></a>
## Writing CSS

The Laravel `package.json` file includes the `bootstrap-sass` package to help you get started prototyping your application's frontend using Bootstrap. However, feel free to add or remove packages from the `package.json` file as needed for your own application. You are not required to use the Bootstrap framework to build your Laravel application - it is simply provided as a good starting point for those who choose to use it.

Before compiling your CSS, install your project's frontend dependencies using NPM:

    npm install

Once the dependencies have been installed using `npm install`, you can compile your SASS files to plain CSS using [Gulp](http://gulpjs.com/). The `gulp` command will process the instructions in your `gulpfile.js` file. Typically, your compiled CSS will be placed in the `public/css` directory:

    gulp

The default `gulpfile.js` included with Laravel will compile the `resources/assets/sass/app.scss` SASS file. This `app.scss` file imports a file of SASS variables and loads Bootstrap, which provides a good starting point for most applications. Feel free to customize the `app.scss` file however you wish or even use an entirely different pre-processor by [configuring Laravel Elixir](docs/{{language}}/{{version}}/elixir).

<a name="writing-javascript"></a>
## Writing JavaScript

All of the JavaScript dependencies required by your application can be found in the `package.json` file in the project's root directory. This file is similar to a `composer.json` file except it specifies JavaScript dependencies instead of PHP dependencies. You can install these dependencies using the [Node package manager (NPM)](https://npmjs.org):

    npm install

By default, the Laravel `package.json` file includes a few packages such as `vue` and `vue-resource` to help you get started building your JavaScript application. Feel free to add or remove from the `package.json` file as needed for your own application.

Once the packages are installed, you can use the `gulp` command to [compile your assets](docs/{{language}}/{{version}}/elixir). Gulp is a command-line build system for JavaScript. When you run the `gulp` command, Gulp will execute the instructions in your `gulpfile.js` file:

    gulp

By default, the Laravel `gulpfile.js` file compiles your SASS and the `resources/assets/js/app.js` file. Within the `app.js` file you may register your Vue components or, if you prefer a different framework, configure your own JavaScript application. Your compiled JavaScript will typically be placed in the `public/js` directory.

> {tip} The `app.js` file will load the `resources/assets/js/bootstrap.js` file which bootstraps and configures Vue, Vue Resource, jQuery, and all other JavaScript dependencies. If you have additional JavaScript dependencies to configure, you may do so in this file.

<a name="writing-vue-components"></a>
### Writing Vue Components

By default, fresh Laravel applications contain an `Example.vue` Vue component located in the `resources/assets/js/components` directory. The `Example.vue` file is an example of a [single file Vue component](https://vuejs.org/guide/application.html#Single-File-Components) which defines its JavaScript and HTML template in the same file. Single file components provide a very convenient approach to building JavaScript driven applications. The example component is registered in your `app.js` file:

    Vue.component('example', require('./components/Example.vue'));

To use the component in your application, you may simply drop it into one of your HTML templates. For example, after running the `make:auth` Artisan command to scaffold your application's authentication and registration screens, you could drop the component into the `home.blade.php` Blade template:

    @extends('layouts.app')

    @section('content')
        <example></example>
    @endsection

> {tip} Remember, you should run the `gulp` command each time you change a Vue component. Or, you may run the `gulp watch` command to monitor and automatically recompile your components each time they are modified.

Of course, if you are interested in learning more about writing Vue components, you should read the [Vue documentation](http://vuejs.org/guide/), which provides a thorough, easy-to-read overview of the entire Vue framework.
