# Laravel Homestead

- [前言](#introduction)
- [安装 & 设置](#installation-and-setup)
    - [第一步](#first-steps)
    - [配置 Homestead](#configuring-homestead)
    - [启动 Vagrant](#launching-the-vagrant-box)
    - [分配到项目](#per-project-installation)
    - [安装 MariaDB](#installing-mariadb)
- [常用方法](#daily-usage)
    - [全局访问 Homestead](#accessing-homestead-globally)
    - [通过 SSH 连接](#connecting-via-ssh)
    - [连接到数据库](#connecting-to-databases)
    - [添加额外的站点](#adding-additional-sites)
    - [配置 Cron 日程](#configuring-cron-schedules)
    - [端口](#ports)
- [网络接口](#network-interfaces)

<a name="introduction"></a>
## 前言


Laravel 致力于让 PHP 的开发过程变得更加轻松愉快，这其中也包含你的本地开发环境。[Vagrant](http://vagrantup.com)提供了一个简单、优雅的方式来管理和配置虚拟机。


Laravel Homestead 是一个官方预封装的 Vagrant box，致力于提供给你一个完美的开发环境，你无需在本机电脑上安装 PHP、HHVM、Web 服务器或其它服务器软件。并且不用再担心系统被搞乱！Vagrant box 为你搞定一切。如果有什么地方出错了，你也可以在几分钟内快速的销毁并重建虚拟机！

Homestead 可以在 Windows、Mac 或 Linux 系统上面运行，里面包含了 Nginx Web 服务器、PHP 7.0、MySQL、Postgres、Redis、Memcached、Node，以及所有你在使用 Laravel 开发时所需要用到的各种软件。

> {note} 如果你是 Windows 用户，你可能需要启用硬件虚拟化（VT-x）。这通常需要通过 BIOS 来启用它。如果你在一个 UEFI 系统上使用 Hyper-V，你可能需要为了访问 VT-x 来禁用 Hyper-V。

<a name="included-software"></a>
### 引入的软件

- Ubuntu 16.04
- Git
- PHP 7.0
- HHVM
- Nginx
- MySQL
- MariaDB
- Sqlite3
- Postgres
- Composer
- Node (With PM2, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd

<a name="installation-and-setup"></a>
## 安装 & 设置

<a name="first-steps"></a>
### 第一步


在你启动你的 Homestead 环境之前，你必须安装 [VirtualBox 5.x](https://www.virtualbox.org/wiki/Downloads) 或者 [VMWare](http://www.vmware.com)，以及 [Vagrant](http://www.vagrantup.com/downloads.html)。所有这些软件在主流操作系统中都提供有简单易用的安装包。

如果使用 VMware provider，你需要同时购买 VMware Fusion / Workstation 以及 [VMware Vagrant plug-in](http://www.vagrantup.com/vmware) 的软件授权。尽管它不是免费的，但是 VMware 可以在共享文件夹上提供更快的性能。

#### 安装 Homestead Vagrant Box

当 VirtuanlBox / VMware 和 Vagrant 安装完毕之后，你可以使用终端执行下面的命令来添加 `laravel/homestead` box 到你的 Vagrant 中。这会花费一些时间来下载这个 box，具体的下载时长依据你当前的网络环境：

    vagrant box add laravel/homestead

如果这个命令失败了，请确保你的 Vagrant 已经是最新的版本。

#### 安装 Homestead

你可以通过简单的克隆代码仓库来安装 Homestead。建议将代码仓库克隆至 "home" 目录中的 `Homestead` 文件夹，如此一来 Homestead box 就能将主机服务提供给你所有的 Laravel 项目：

    cd ~

    git clone https://github.com/laravel/homestead.git Homestead


一旦你克隆完 Homestead 的代码仓库，即可在 Homestead 目录中运行 `bash init.sh` 命令来创建 `Homestead.yaml` 配置文件。Homestead.yaml 文件将会被放置在你的 `~/.homestead` 目录中：

    bash init.sh

<a name="configuring-homestead"></a>
### 配置 Homestead

#### 设置你的驱动

`~/.homestead/Homestead.yaml` 文件中的 `provider` 键表明哪种 Vagrant 虚拟机驱动将被使用：`virtualbox`，`vmware_fusion`，或者 `vmware_workstation`。你可以依据自己的喜好来进行设置：

    provider: virtualbox

#### 配置共享目录

你可以在 `Homestead.yaml` 文件中 `folders` 属性里列出所有你希望共享到你的 Homestead 环境中的文件夹。当这些文件夹中的文件改变时，它们将会在你的本地机器和 Homestead 环境中保持同步。你可以在需要时在这里设置多个共享文件夹:

    folders:
        - map: ~/Code
          to: /home/vagrant/Code

如果需要开启 [NFS](http://docs.vagrantup.com/v2/synced-folders/nfs.html), 那么你需要在同步的文件夹中设置如下配置：

    folders:
        - map: ~/Code
          to: /home/vagrant/Code
          type: "nfs"

#### 配置 Nginx 站点


对 Nginx 不熟悉？没关系。`site` 属性允许你轻松的在你的 Homestead 环境中映射 "域名" 和文件夹。在 `Homestead.yaml` 文件中已经包含了一个简单的站点配置。这次，你可以按需的在你的 Homestead 环境中添加多个站点。Homestead 可以为你所有的正在进行中的项目提供便捷的虚拟化环境：

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public

你可以通过设置 `hhvm` 选项为 `true` 让任意的 Homestead 站点使用 [HHVM](http://hhvm.com):

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          hhvm: true

如果你在供应了 Homestead box 之后修改了 `sites` 属性，那么你需要重新运行 `vagrant reload --provision` 命令来更新虚拟机中的 Nginx 配置。

#### Hosts 文件


你必须在你主机里的 `hosts` 文件中添加 Nginx 站点所需要的 "域名"。`hosts` 文件将会重定向请求到你的 Homestead 虚拟机里的 Homestead 站点。在 Mac 和 Linux 系统中，这个文件被存放在 `/etc/hosts`。在 Windows 中，它被存放在 `C:\Windows\System32\drivers\etc\hosts`。你需要在该文件中添加像下面的一行：

    192.168.10.10  homestead.app

你需要确保 IP 地址和你的 `~/.homestead/Homestead.yaml` 文件中设置的一致。一旦你添加了域名到你的 `hosts` 文件并且启动完成 Vargrant box，你就可以通过你的浏览器访问该站点:

    http://homestead.app

<a name="launching-the-vagrant-box"></a>
### 启动 Vagrant Box

当你按需的编辑完成 `Homestead.yaml`，你就可以从你的 Homestead 目录中运行 `vagrant up` 命令。Vagrant 将会启动你的虚拟机并且自动的分配共享目录和 Nginx 站点。

你也可以通过使用 `vagrant destroy --force` 命令来销毁虚拟机。

<a name="per-project-installation"></a>
### 分配到项目

除了将 Homestead 安装成全局环境，让所有的项目共用一个 Homestead box。你也可以为每一个你所管理的项目进行独立的配置 Homestead。为每一个项目独立的配置 Homestead 你就可以将 `Vagrantfile` 关联到项目中，这有益于其他开发者直接使用 `vagrant up` 来进行工作。

你可以通过 Composer 直接将 Homestead 安装到项目中:

    composer require laravel/homestead --dev

当 Homestead 安装完成之后，你就可以使用 `make` 命令来生成 `Vagrantfile` 和 `Homestead.yaml` 文件，它们会直接生成在项目的根目录中。`make` 命令会自动的配置 `Homestead.yaml` 文件中的 `sites` 和 `folders` 指令。

Mac / Linux:

    php vendor/bin/homestead make

Windows:

	vendor\\bin\\homestead make


接着，你就可以运行 `vagrant up` 命令来启动虚拟机，然后通过浏览器访问 `http://homestead.app` 站点了。记住，你还是需要在 `/etc/hosts` 文件中写入 `homestead.app` 的 ip 地址，或者是录入你自由选择的域名。

<a name="installing-mariadb"></a>
### 安装 MariaDB

如果你喜欢使用 MariaDB 取代 MySQL，那么你可以在你的 `Homestead.yaml` 文件中添加 `mariadb` 选项。这个选项会删除 MySQL，并且安装 MariaDB。由于 MariaDB 是从 MySQL 中分离出来的，所以你仍可以在你的应用数据库配置中使用 `mysql` 数据库作为驱动：

    box: laravel/homestead
    ip: "192.168.20.20"
    memory: 2048
    cpus: 4
    provider: virtualbox
    mariadb: true

<a name="daily-usage"></a>
## 常用方法

<a name="accessing-homestead-globally"></a>
### 全局访问 Homestead

有时候你可能希望在你的文件系统中的任意位置都可以使用 `vagrant up` 来启动你的 Homestead。你可以通过在你的 Bash 文件中添加一个简单的 Bash 方法来做到。这个方法会允许你在系统的任意位置执行任意的 Vagrant 命令，并且它会自动的将命令输入到你的 Homestead 中：

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

你需要确保函数中的 `~/Homestead` 路径调整为你实际安装的 Homestead 路径一致。完成这个方法之后，你就可以在系统中的任意位置执行类似 `homestead up` 或者 `homestead ssh` 命令。

<a name="connecting-via-ssh"></a>
### 通过 SSH 连接

你可以在你的 Homestead 目录下使用终端命令 `vagrant ssh` 来连接到你的虚拟中。

但是，你可能会经常需要使用 SSH 来连接你的 Homestead，你可以考虑添加类似上面描述的 "function" 来快速的 SSH 到你的 Homestead。

<a name="connecting-to-databases"></a>
### 连接到数据库

在 `homestead` 中已经预装了 MySQL 和 Postgres 数据库。为了方便使用，Laravel 的 `.env` 文件中已经对这些数据库的配置提供了支持。

如果你想要在主机中使用 Navicat 或者 Sequel Pro 来访问虚拟机中的 MySQL 或者 Postgres 数据库，你应该连接到 `127.0.0.1` 的 `33060`（MySQL）端口或者 `54320`（Postgres）端口。它们的用户名和密码都是 `homestead` / `secret`。

> {note} 你应该在主机中使用这些非标准的端口来连接虚拟机中的数据库。这是由于，虚拟主机中的 Laravel 在运行时，依然使用的是配置文件中的 3306 和 5432 端口。

<a name="adding-additional-sites"></a>
### 添加额外的站点

当你的 Homestead 环境配置完毕且成功运行后，你可能希望为你的 Laravel 应用添加额外的 Nginx 站点。你可以在单个 Homestead 环境中运行多个 Laravel 安装程序。你可以在 `~/.homestead/Homestead.yaml` 文件中添加站点之后，直接的在你的 Homestead 目录中运行 `vagrant provision` 命令。

<a name="configuring-cron-schedules"></a>
### 配置 Cron 调度器

Laravel 提供了一种便利的方式来完成 [调度 Cron 任务](/docs/{{language}}/{{version}}/scheduling)。通过 `schedule:run` Artisan 命令，调度便会在每分钟被运行。`schedule:run` 命令会检查 `App\Console\Kernel` 类中所定义的调度任务，并判断哪个任务需要被运行。

如果你希望在 Homestead 站点中执行 `schedule:run` 命令。你可以在站点配置选项中设置 `schedule` 选项为 `true`:

    sites:
        - map: homestead.app
          to: /home/vagrant/Code/Laravel/public
          schedule: true

站点中的 Cron 任务会被定义在虚拟机的 `/etc/cron.d` 目录中。

<a name="ports"></a>
### 端口

默认的，下面的端口会被转发到你的 Homestead 环境中：

- **SSH:** 2222 &rarr; 转发到 22
- **HTTP:** 8000 &rarr; 转发到 80
- **HTTPS:** 44300 &rarr; 转发到 443
- **MySQL:** 33060 &rarr; 转发到 3306
- **Postgres:** 54320 &rarr; 转发到 5432

#### 转发额外的端口

如果你需要，你可以转发额外的端口到你的 Vagrant box 中，你也可以指定转发的协议：

    ports:
        - send: 93000
          to: 9300
        - send: 7777
          to: 777
          protocol: udp

<a name="network-interfaces"></a>
## 网络接口

`Homestead.yaml` 使用配置文件中的 `networks` 属性用来配置 Homestead 环境中的网络通讯接口。你可以按需的进行配置多个接口：

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

你可以通过设置配置中的 `bridge` 选项和修改网络类型为 `public_network` 来启用 [bridged](https://www.vagrantup.com/docs/networking/public_network.html):

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

如果需要开启 [DHCP](https://www.vagrantup.com/docs/networking/public_network.html)，你只需要从你的配置文件中删除 `ip` 选项即可：

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"
