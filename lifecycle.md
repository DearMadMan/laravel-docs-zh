# 请求生命周期


- [前言](#introduction)
- [生命周期概述](#lifecycle-overview)
- [聚焦服务提供者](#focus-on-service-providers)

<a name="introduction"></a>
## 前言

在现实中，如果你明白了你所使用的工具的工作原理，你就会在时候的时候感觉非常的舒适与自信。开发应用并没有什么什么不同，当你弄懂了这些开发工具功能，你就会在使用时得心应收。

这份文档会深层次的带你浏览 Laravel 框架是如何工作的。通过对框架更全面的了解，一切都会显得不再那么神秘，你会更加自信的去构建应用。

如果你并不能理解该文档的所有内容，请不要伤心，你需要先试着掌握一些基本的概念，你的知识体系会随着对文档的探索而增长。

<a name="lifecycle-overview"></a>
## 生命周期概述

### 第一件事

Laravel 应用中的所有请求的入口都是 `public/index.php` 文件，所有的请求都会被导向该文件。`index.php` 文件中并没有存储太多的代码，相反，它只是用于装载框架的其余部分的起始点。

`index.php` 文件会加载 Composer 生成的自动加载器的配置信息，然后从 `/bootstrap/app.php` 文件中加载 Laravel 的应用实例，Laravel 的第一个动作就是创建一个 [服务容器](/docs/{{language}}/{{version}}/container) 的实例。

### HTTP / 控制台内核

根据请求进入应用的类型，请求会被分配到 HTTP 内核或者 Console 内核处理。这两个内核都会作为所有请求流进过的中心处理器。现在，让我们只聚焦在 HTTP 内核上，它被存储在 `app/http/Kernel.php` 文件中。

HTTP 内核继承自 `Illuminate\Foundation\Http\Kernel` 类，这个类定义了一个 `bootstrappers` 的数组，这些类会在请求处理前运行，这些 `bootstrappers` 执行错误处理，日志，[探测应用环境](/docs/{{language}}/{{version}}/installation#environment-configuration) 和履行一些其它请求被处理前执行的任务。

HTTP 内核也定义列一系列的 HTTP [中间件](/docs/{{language}}/{{version}}/middleware)，所有的请求在被处理前都会经过这些中间件。这些中间件包括了 [HTTP session](/docs/{{language}}/{{version}}/session) 的读写，[verifying the CSRF token](/docs/{{language}}/{{version}}/routing#csrf-protection)，等等。

HTTP 内核的 `handle` 方法的签证非常简单：接收一个 `Request` 返回一个 `Response`。你可以把这个核心概念想象成一个黑盒子，左边 HTTP 请求进去，右边返回 HTTP 响应。

#### 服务提供者

启动内核中最重要的一步就是为你的应用加载 [服务提供者](/docs/{{language}}/{{version}}/providers)。所有的服务提供者都在 `config/app.php` 文件的 `providers` 数组中进行配置。首先，所有经过配置的提供者都会执行其自身的 `register` 方法，然后当所有的提供者都完成注册之后，才会陆续的调用 `boot` 方法。

服务提供者主要负责启动框架中的各个组件，比如数据库组件、队列、验证和路由组件。正是由于框架中的各种核心功能都是这些组件所提供的，所以服务提供者是整个框架启动中最重要的一环。

#### 分发请求

当应用启动完毕，并且所有的服务提供者都完成注册之后，`Request` 将会移交到路由进行分发。路由器将会分发该请求到路由或控制器中，同时也会激活分配到相应路由或控制器的中间件。

<a name="focus-on-service-providers"></a>
## 聚焦服务提供者

服务提供者是 Laravel 应用能够成功启动的最关键的部分。首先创建应用的实例,接着服务提供者进行注册，然后处理已经启动成功的应用请求，整个流程就是这么简单!

能够深刻理解并掌握 Laravel 通过服务提供者进行启动和构建应用的过程是极其有宝贵的。当然，你的应用中默认的服务提供者都已经被存储在 `app/Providers` 目录中。

默认的，`AppServiceProvider` 是一个相当简单的类。这里是你在应用中添加自己的启动项和做服务容器绑定的好去处。当然，在大型应用中，你可以自行构建服务提供者，以使每一个提供者都具有单一的职责。
