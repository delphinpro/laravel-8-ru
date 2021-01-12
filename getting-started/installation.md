# Установка

## Знакомьтесь, Laravel

Laravel — это фреймворк веб-приложений с выразительным, элегантным синтаксисом. Веб-фреймворк предоставляет структуру и отправную точку для создания вашего приложения, позволяя вам сосредоточиться на создании чего-то удивительного, в то время как мы потеем над деталями.

Laravel стремится обеспечить уникальные возможности для разработчиков, предоставляя такие мощные функции, как продуманное внедрение зависимостей, уровень абстракции базы данных, очереди и запланированные задания, тестирование модулей и интеграции, и многое другое.

Независимо от того, являетесь ли вы новичком в PHP или веб-фреймворках или имеете многолетний опыт работы, Laravel — это фреймворк, который может расти вместе с вами. Мы поможем вам сделать первые шаги в качестве веб-разработчика или поднимем ваш опыт на новый уровень. Мы с нетерпением ожидаем увидеть, что вы создадите.

### Почему Laravel?

При создании веб-приложения вам доступны различные инструменты и фреймворки. Но, тем не менее, мы считаем, что Laravel является лучшим выбором для создания современных веб-приложений с полным стеком.

#### Прогрессивный фреймворк

Нам нравится называть Laravel "прогрессивным" фреймворком. Под этим мы подразумеваем, что Laravel растет вместе с вами. Если вы только делаете первые шаги в веб-разработке, обширная библиотека документации, руководств и [видео-уроков](https://laracasts.com/) Laravel поможет вам изучать основы, не перенапрягаясь.

Если вы senior, Laravel предоставляет вам надежные инструменты для [внедрения зависимостей](../service-container.md), [модульного тестирования](../testing-1.md), [очередей](../queues.md), [событий в реальном времени](../translyaciya.md), и многое другое. Laravel тонко настроен для создания профессиональных веб-приложений и готов к обработке корпоративных рабочих нагрузок.

#### A Scalable Framework

Laravel is incredibly scalable. Thanks to the scaling-friendly nature of PHP and Laravel's built-in support for fast, distributed cache systems like Redis, horizontal scaling with Laravel is a breeze. In fact, Laravel applications have been easily scaled to handle hundreds of millions of requests per month.

Need extreme scaling? Platforms like [Laravel Vapor](https://vapor.laravel.com/) allow you to run your Laravel application at nearly limitless scale on AWS's latest serverless technology.

#### A Community Framework

Laravel combines the best packages in the PHP ecosystem to offer the most robust and developer friendly framework available. In addition, thousands of talented developers from around the world have [contributed to the framework](https://github.com/laravel/framework). Who knows, maybe you'll even become a Laravel contributor.

## Your First Laravel Project

We want it to be as easy as possible to get started with Laravel. There are a variety of options for developing and running a Laravel project on your own computer. While you may wish to explore these options at a later time, Laravel provides [Sail](https://laravel.com/docs/8.x/sail), a built-in solution for running your Laravel project using [Docker](https://www.docker.com/).

Docker is a tool for running applications and services in small, light-weight "containers" which do not interfere with your local computer's installed software or configuration. This means you don't have to worry about configuring or setting up complicated development tools such as web servers and databases on your personal computer. To get started, you only need to install [Docker Desktop](https://www.docker.com/products/docker-desktop).

Laravel Sail is a light-weight command-line interface for interacting with Laravel's default Docker configuration. Sail provides a great starting point for building a Laravel application using PHP, MySQL, and Redis without requiring prior Docker experience.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Already a Docker expert? Don't worry! Everything about Sail can be customized using the `docker-compose.yml` file included with Laravel.

### Getting Started On macOS

If you're developing on a Mac and [Docker Desktop](https://www.docker.com/products/docker-desktop) is already installed, you can use a simple terminal command to create a new Laravel project. For example, to create a new Laravel application in a directory named "example-app", you may run the following command in your terminal:

```text
curl -s https://laravel.build/example-app | bash
```

Of course, you can change "example-app" in this URL to anything you like. The Laravel application's directory will be created within the directory you execute the command from.

After the project has been created, you can navigate to the application directory and start Laravel Sail. Laravel Sail provides a simple command-line interface for interacting with Laravel's default Docker configuration:

```text
cd example-app

./vendor/bin/sail up
```

The first time you run the Sail `up` command, Sail's application containers will be built on your machine. This could take several minutes. **Don't worry, subsequent attempts to start Sail will be much faster.**

Once the application's Docker containers have been started, you can access the application in your web browser at: [http://localhost](http://localhost/).

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> To continue learning more about Laravel Sail, review its [complete documentation](https://laravel.com/docs/8.x/sail).

### Getting Started On Windows

Before we create a new Laravel application on your Windows machine, make sure to install [Docker Desktop](https://www.docker.com/products/docker-desktop). Next, you should ensure that Windows Subsystem for Linux 2 \(WSL2\) is installed and enabled. WSL allows you to run Linux binary executables natively on Windows 10. Information on how to install and enable WSL2 can be found within Microsoft's [developer environment documentation](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> After installing and enabling WSL2, you should ensure that Docker Desktop is [configured to use the WSL2 backend](https://docs.docker.com/docker-for-windows/wsl/).

Next, you are ready to create your first Laravel project. Launch [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab) and begin a new terminal session for your WSL2 Linux operating system. Next, you can use a simple terminal command to create a new Laravel project. For example, to create a new Laravel application in a directory named "example-app", you may run the following command in your terminal:

```text
curl -s https://laravel.build/example-app | bash
```

Of course, you can change "example-app" in this URL to anything you like. The Laravel application's directory will be created within the directory you execute the command from.

After the project has been created, you can navigate to the application directory and start Laravel Sail. Laravel Sail provides a simple command-line interface for interacting with Laravel's default Docker configuration:

```text
cd example-app

./vendor/bin/sail up
```

The first time you run the Sail `up` command, Sail's application containers will be built on your machine. This could take several minutes. **Don't worry, subsequent attempts to start Sail will be much faster.**

Once the application's Docker containers have been started, you can access the application in your web browser at: [http://localhost](http://localhost/).

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> To continue learning more about Laravel Sail, review its [complete documentation](https://laravel.com/docs/8.x/sail).

#### Developing Within WSL2

Of course, you will need to be able to modify the Laravel application files that were created within your WSL2 installation. To accomplish this, we recommend using Microsoft's [Visual Studio Code](https://code.visualstudio.com/) editor and their first-party extension for [Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack).

Once these tools are installed, you may open any Laravel project by executing the `code .` command from your application's root directory using Windows Terminal.

### Getting Started On Linux

If you're developing on Linux and [Docker](https://www.docker.com/) is already installed, you can use a simple terminal command to create a new Laravel project. For example, to create a new Laravel application in a directory named "example-app", you may run the following command in your terminal:

```text
curl -s https://laravel.build/example-app | bash
```

Of course, you can change "example-app" in this URL to anything you like. The Laravel application's directory will be created within the directory you execute the command from.

After the project has been created, you can navigate to the application directory and start Laravel Sail. Laravel Sail provides a simple command-line interface for interacting with Laravel's default Docker configuration:

```text
cd example-app

./vendor/bin/sail up
```

The first time you run the Sail `up` command, Sail's application containers will be built on your machine. This could take several minutes. **Don't worry, subsequent attempts to start Sail will be much faster.**

Once the application's Docker containers have been started, you can access the application in your web browser at: [http://localhost](http://localhost/).

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> To continue learning more about Laravel Sail, review its [complete documentation](https://laravel.com/docs/8.x/sail).

### Installation Via Composer

If your computer already has PHP and Composer installed, you may create a new Laravel project by using Composer directly. After the application has been created, you may start Laravel's local development server using the Artisan CLI's `serve` command:

```text
composer create-project laravel/laravel example-app

cd example-app

php artisan serve
```

#### The Laravel Installer

Or, you may install the Laravel Installer as a global Composer dependency:

```text
composer global require laravel/installer

laravel new example-app

php artisan serve
```

Make sure to place Composer's system-wide vendor bin directory in your `$PATH` so the `laravel` executable can be located by your system. This directory exists in different locations based on your operating system; however, some common locations include:

* macOS: `$HOME/.composer/vendor/bin`
* Windows: `%USERPROFILE%\AppData\Roaming\Composer\vendor\bin`
* GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin` or `$HOME/.composer/vendor/bin`

## Initial Configuration

All of the configuration files for the Laravel framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

Laravel needs almost no additional configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

#### Environment Based Configuration

Since many of Laravel's configuration option values may vary depending on whether your application is running on your local computer or on a production web server, many important configuration values are defined using the `.env` file that exists at the root of your application.

Your `.env` file should not be committed to your application's source control, since each developer / server using your application could require a different environment configuration. Furthermore, this would be a security risk in the event an intruder gains access to your source control repository, since any sensitive credentials would get exposed.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> For more information about the `.env` file and environment based configuration, check out the full [configuration documentation](https://laravel.com/docs/8.x/configuration#environment-configuration).

## Next Steps

Now that you have created your Laravel project, you may be wondering what to learn next. First, we strongly recommend becoming familiar with how Laravel works by reading the following documentation:

* [Request Lifecycle](https://laravel.com/docs/8.x/lifecycle)
* [Configuration](https://laravel.com/docs/8.x/configuration)
* [Directory Structure](https://laravel.com/docs/8.x/structure)
* [Service Container](https://laravel.com/docs/8.x/container)
* [Facades](https://laravel.com/docs/8.x/facades)

How you want to use Laravel will also dictate the next steps on your journey. There are a variety of ways to use Laravel, and we'll explore two primary use cases for the framework below.

### Laravel The Full Stack Framework

Laravel may serve as a full stack framework. By "full stack" framework we mean that you are going to use Laravel to route requests to your application and render your frontend via [Blade templates](https://laravel.com/docs/8.x/blade) or using a single-page application hybrid technology like [Inertia.js](https://inertiajs.com/). This is the most common way to use the Laravel framework.

If this is how you plan to use Laravel, you may want to check out our documentation on [routing](https://laravel.com/docs/8.x/routing), [views](https://laravel.com/docs/8.x/views), or the [Eloquent ORM](https://laravel.com/docs/8.x/eloquent). In addition, you might be interested in learning about community packages like [Livewire](https://laravel-livewire.com/) and [Inertia.js](https://inertiajs.com/). These packages allow you to use Laravel as a full-stack framework while enjoying many of the UI benefits provided by single-page JavaScript applications.

If you are using Laravel as a full stack framework, we also strongly encourage you to learn how to compile your application's CSS and JavaScript using [Laravel Mix](https://laravel.com/docs/8.x/mix).

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> If you want to get a head start building your application, check out one of our official [application starter kits](https://laravel.com/docs/8.x/starter-kits).

### Laravel The API Backend

Laravel may also serve as an API backend to a JavaScript single-page application or mobile application. For example, you might use Laravel as an API backend for your [Next.js](https://nextjs.org/) application. In this context, you may use Laravel to provide [authentication](https://laravel.com/docs/8.x/sanctum) and data storage / retrieval for your application, while also taking advantage of Laravel's powerful services such as queues, emails, notifications, and more.

If this is how you plan to use Laravel, you may want to check out our documentation on [routing](https://laravel.com/docs/8.x/routing), [Laravel Sanctum](https://laravel.com/docs/8.x/sanctum), and the [Eloquent ORM](https://laravel.com/docs/8.x/eloquent).

