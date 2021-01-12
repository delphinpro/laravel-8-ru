# Конфигурация

## [Introduction](https://laravel.com/docs/8.x/configuration#introduction)

All of the configuration files for the Laravel framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

These configuration files allow you to configure things like your database connection information, your mail server information, as well as various other core configuration values such as your application timezone and encryption key.

## Конфигурация окружения <a id="env-config"></a>

It is often helpful to have different configuration values based on the environment where the application is running. For example, you may wish to use a different cache driver locally than you do on your production server.

To make this a cinch, Laravel utilizes the [DotEnv](https://github.com/vlucas/phpdotenv) PHP library. In a fresh Laravel installation, the root directory of your application will contain a `.env.example` file that defines many common environment variables. During the Laravel installation process, this file will automatically be copied to `.env`.

Laravel's default `.env` file contains some common configuration values that may differ based on whether your application is running locally or on a production web server. These values are then retrieved from various Laravel configuration files within the `config` directory using Laravel's `env` function.

If you are developing with a team, you may wish to continue including a `.env.example` file with your application. By putting placeholder values in the example configuration file, other developers on your team can clearly see which environment variables are needed to run your application.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Any variable in your `.env` file can be overridden by external environment variables such as server-level or system-level environment variables.

#### **Environment File Security**

Your `.env` file should not be committed to your application's source control, since each developer / server using your application could require a different environment configuration. Furthermore, this would be a security risk in the event an intruder gains access to your source control repository, since any sensitive credentials would get exposed.

### [Environment Variable Types](https://laravel.com/docs/8.x/configuration#environment-variable-types)

All variables in your `.env` files are typically parsed as strings, so some reserved values have been created to allow you to return a wider range of types from the `env()` function:

| `.env` Value | `env()` Value |
| :--- | :--- |
| true | \(bool\) true |
| \(true\) | \(bool\) true |
| false | \(bool\) false |
| \(false\) | \(bool\) false |
| empty | \(string\) '' |
| \(empty\) | \(string\) '' |
| null | \(null\) null |
| \(null\) | \(null\) null |

If you need to define an environment variable with a value that contains spaces, you may do so by enclosing the value in double quotes:

```php
APP_NAME="My Application"
```

### [Retrieving Environment Configuration](https://laravel.com/docs/8.x/configuration#retrieving-environment-configuration)

All of the variables listed in this file will be loaded into the `$_ENV` PHP super-global when your application receives a request. However, you may use the `env` helper to retrieve values from these variables in your configuration files. In fact, if you review the Laravel configuration files, you will notice many of the options are already using this helper:

```php
'debug' => env('APP_DEBUG', false),
```

The second value passed to the `env` function is the "default value". This value will be returned if no environment variable exists for the given key.

### [Determining The Current Environment](https://laravel.com/docs/8.x/configuration#determining-the-current-environment)

The current application environment is determined via the `APP_ENV` variable from your `.env` file. You may access this value via the `environment` method on the `App` [facade](https://laravel.com/docs/8.x/facades):

```php
use Illuminate\Support\Facades\App;

$environment = App::environment();
```

You may also pass arguments to the `environment` method to determine if the environment matches a given value. The method will return `true` if the environment matches any of the given values:

```php
if (App::environment('local')) {
    // The environment is local
}

if (App::environment(['local', 'staging'])) {
    // The environment is either local OR staging...
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> The current application environment detection can be overridden by defining a server-level `APP_ENV` environment variable.

## [Accessing Configuration Values](https://laravel.com/docs/8.x/configuration#accessing-configuration-values)

You may easily access your configuration values using the global `config` helper function from anywhere in your application. The configuration values may be accessed using "dot" syntax, which includes the name of the file and option you wish to access. A default value may also be specified and will be returned if the configuration option does not exist:

```php
$value = config('app.timezone');

// Retrieve a default value if the configuration value does not exist...
$value = config('app.timezone', 'Asia/Seoul');
```

To set configuration values at runtime, pass an array to the `config` helper:

```php
config(['app.timezone' => 'America/Chicago']);
```

## [Configuration Caching](https://laravel.com/docs/8.x/configuration#configuration-caching)

To give your application a speed boost, you should cache all of your configuration files into a single file using the `config:cache` Artisan command. This will combine all of the configuration options for your application into a single file which can be quickly loaded by the framework.

You should typically run the `php artisan config:cache` command as part of your production deployment process. The command should not be run during local development as configuration options will frequently need to be changed during the course of your application's development.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> If you execute the `config:cache` command during your deployment process, you should be sure that you are only calling the `env` function from within your configuration files. Once the configuration has been cached, the `.env` file will not be loaded and all calls to the `env` function will return `null`.

## [Debug Mode](https://laravel.com/docs/8.x/configuration#debug-mode)

The `debug` option in your `config/app.php` configuration file determines how much information about an error is actually displayed to the user. By default, this option is set to respect the value of the `APP_DEBUG` environment variable, which is stored in your `.env` file.

For local development, you should set the `APP_DEBUG` environment variable to `true`. **In your production environment, this value should always be `false`. If the variable is set to `true` in production, you risk exposing sensitive configuration values to your application's end users.**

## [Maintenance Mode](https://laravel.com/docs/8.x/configuration#maintenance-mode)

When your application is in maintenance mode, a custom view will be displayed for all requests into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A maintenance mode check is included in the default middleware stack for your application. If the application is in maintenance mode, a `MaintenanceModeException` will be thrown with a status code of 503.

To enable maintenance mode, execute the `down` Artisan command:

```bash
php artisan down
```

You may also provide a `retry` option to the `down` command, which will be set as the `Retry-After` HTTP header's value:

```bash
php artisan down --retry=60
```

**Bypassing Maintenance Mode**

Even while in maintenance mode, you may use the `secret` option to specify a maintenance mode bypass token:

```bash
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

After placing the application in maintenance mode, you may navigate to the application URL matching this token and Laravel will issue a maintenance mode bypass cookie to your browser:

```text
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

When accessing this hidden route, you will then be redirected to the `/` route of the application. Once the cookie has been issued to your browser, you will be able to browse the application normally as if it was not in maintenance mode.

**Pre-Rendering The Maintenance Mode View**

If you utilize the `php artisan down` command during deployment, your users may still occasionally encounter errors if they access the application while your Composer dependencies or other infrastructure components are updating. This occurs because a significant part of the Laravel framework must boot in order to determine your application is in maintenance mode and render the maintenance mode view using the templating engine.

For this reason, Laravel allows you to pre-render a maintenance mode view that will be returned at the very beginning of the request cycle. This view is rendered before any of your application's dependencies have loaded. You may pre-render a template of your choice using the `down` command's `render` option:

```bash
php artisan down --render="errors::503"
```

**Redirecting Maintenance Mode Requests**

While in maintenance mode, Laravel will display the maintenance mode view for all application URLs the user attempts to access. If you wish, you may instruct Laravel to redirect all requests to a specific URL. This may be accomplished using the `redirect` option. For example, you may wish to redirect all requests to the `/` URI:

```bash
php artisan down --redirect=/
```

**Disabling Maintenance Mode**

To disable maintenance mode, use the `up` command:

```bash
php artisan up
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> You may customize the default maintenance mode template by defining your own template at `resources/views/errors/503.blade.php`.

**Maintenance Mode & Queues**

While your application is in maintenance mode, no [queued jobs](https://laravel.com/docs/8.x/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.

**Alternatives To Maintenance Mode**

Since maintenance mode requires your application to have several seconds of downtime, consider alternatives like [Laravel Vapor](https://vapor.laravel.com/) and [Envoyer](https://envoyer.io/) to accomplish zero-downtime deployment with Laravel.

