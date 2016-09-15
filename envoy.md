# Envoy 任务执行者

- [前言](#introduction)
    - [安装](#installation)
- [编写任务](#writing-tasks)
    - [引导](#setup)
    - [变量](#variables)
    - [仓储](#stories)
    - [多服务器支持](#multiple-servers)
- [执行任务](#running-tasks)
    - [任务执行确认](#confirming-task-execution)
- [通知](#notifications)
    - [Slack](#slack)

<a name="introduction"></a>
## 前言

[Laravel Envoy](https://github.com/laravel/envoy) 为远端服务器常用任务的定义与执行提供了迷你简洁的语法。你可以通过使用 Blade 语法样式轻松的为部署，Artisan 命令的执行等设置任务。目前，Envoy 只支持 Mac 和 Linux 操作系统。

<a name="installation"></a>
### 安装

首先，你需要通过 Composer 的 `global require` 命令来安装 Envoy：

    composer global require "laravel/envoy=~1.0"

由于全局 Composer 类库有时出于某些原因从而导致扩展包的版本冲突，你可以考虑使用 `cgr`，它就是一种替换 `composer global require` 命令的简单方案。`cgr` 类库的安装指南你可以在 [GitHub](https://github.com/consolidation-org/cgr) 中找到。

> {note} 你需要确保 `~/.composer/vendor/bin` 目录被加入到你的 PATH 中，这样才能使你在使用终端时可以直接使用 `envoy` 命令。

#### 更新 Envoy

你可以使用 Composer 来维持 Envoy 的更新。通过发布 `composer global update` 命令，你可以更新安装在本地的所有全局的 Composer 扩展包：

    composer global update

<a name="writing-tasks"></a>
## 编写任务

所有的 Envoy 任务应该被定义在项目根目录下的 `Envoy.blade.php` 文件中。这里有一个简单的示例：

    @servers(['web' => ['user@192.168.1.1']])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

就如你所看到的，`@servers` 指令被定义在文件的头部，并且包含一个数组，数组中包含服务器的列表。`@task` 指令用来定义任务，它包含一个任务名称，和一个数组参数，数组中包含一个 `on` 键，它的值就是任务所要执行的服务器，它应该是 `@servers` 指令列表中的一个或多个。你应该在 `@task` 指令的内部放置 Bash 代码，这些代码会在任务执行时传递给所要执行的远端服务器。

你可以强制指定服务器的 IP 为 `127.0.0.1` 来执行本地的任务：

    @servers(['localhost' => '127.0.0.1'])

<a name="setup"></a>
### 引导

有时候，你可能希望在执行 Envoy 任务之前先执行某些 PHP 操作。你可以使用 `@setup` 指令来声明变量，并且你可以在其内部使用 PHP 来工作：

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

你也可以在 `Envoy.blade.php` 文件的顶部使用 `@include` 指令来引入任意的外部 PHP 文件：

    @include('vendor/autoload.php')

    @task('foo')
        # ...
    @endtask

<a name="variables"></a>
### 变量

如果你需要的话，你可以使用命令行选项值来传递变量到 Envoy 任务中

    envoy run deploy --branch=master

你可以在你的任务中通过 Blade 的 "echo" 语法使用该选项值。当然，你也可以在你的任务中使用 `if` 语句和循环语句。比如，让我们在执行 `git pull` 命令之前选验证一下是否存在 `$branch` 变量：

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site

        @if ($branch)
            git pull origin {{ $branch }}
        @endif

        php artisan migrate
    @endtask

<a name="stories"></a>
### 仓储

仓储允许你定义一个命令来顺序的执行一组任务。这允许你在一个繁重的任务中将一些小的聚焦的任务集中起来。举个实例，我们定义一个 `deploy` 仓储来执行 `git` 和 `composer` 任务：

    @servers(['web' => '192.168.1.1'])

    @story('deploy')
        git
        composer
    @endstory

    @task('git')
        git pull origin master
    @endtask

    @task('composer')
        composer install
    @endtask

一旦你定义完成了仓储的定义，你就可以通过一条命令来运行多个任务：

    envoy run deploy

<a name="multiple-servers"></a>
### 多服务器支持

Envoy 允许你轻松的跨多个服务器执行任务。首先，你需要在 `@servers` 指令中添加额外的服务器。每个服务器应该被分配一个唯一的名字。当你添加完额外的服务器之后，你需要在待执行的任务指令中使用数组 `on` 选项来列出待执行的服务器：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

#### 并行运行

默认的，任务会在服务器间串行连续的执行，这意味着只有在当前服务器执行任务完成之后才会执行下一个服务器的任务。如果你希望跨服务器并行执行任务。你可以在任务指令中添加 `parallel` 选项：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="running-tasks"></a>
## 执行任务

你需要使用 Envoy 的 `run` 命令来执行 `Envoy.blade.php` 文件中所定义的任务。你可以传递一个任务的名称或者仓储名称到命令中。Envoy 会执行任务并同步显示服务器执行的输出：

    envoy run task

<a name="confirming-task-execution"></a>
### 任务执行确认

如果你希望在远端服务器执行所给定任务之前先进行提示，你可以在你的任务定义时添加 `confirm` 指令。这个选项常用于一些具有破坏性的操作中：

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="notifications"></a>
<a name="hipchat-notifications"></a>
## 通知

<a name="slack"></a>
### Slack

Envoy 也支持在每个任务执行之后发送通知到 [Slack](https://slack.com) 中。`@slack` 指令接收一个 Slack hook URL，一个频道名称。 你可以通过在 Slack 的网站上创建一个 `Incoming WebHooks` 来获取 webhook URL。你应该传递整个 webhook URL 到你的 `@slack` 指令中：

    @after
        @slack('webhook-url', '#bots')
    @endafter

你可以提供以下作为频道的参数之一：

<div class="content-list" markdown="1">
- `#channel`: 发送通知到频道
- `@user`: 发送通知到用户
</div>

