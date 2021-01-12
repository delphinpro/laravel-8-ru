# Fortify

## Вступление <a id="introduction"></a>

Laravel Fortify is a frontend agnostic authentication backend implementation for Laravel. Fortify регистрирует маршруты и контроллеры, необходимые для реализации всех функций аутентификации Laravel, включая вход в систему, регистрацию, сброс пароля, проверку электронной почты и многое другое. После установки Fortify, вы можете запустить команду `route:list` , чтобы увидеть маршруты, которые были зарегистрированы.

Так как Fortify не предоставляет свой собственный пользовательский интерфейс, он предназначен для работы с Вашим собственным интерфейсом, который делает запросы к зарегистрированным маршрутам. О том, как именно делать запросы к этим маршрутам, мы поговорим далее.

{% hint style="info" %}
Помните, Fortify - это пакет, предназначенный для того, чтобы дать вам старт в реализации функций аутентификации в Laravel. **Вы не обязаны его использовать.** Вы всегда можете вручную взаимодействовать со службами аутентификации Laravel, следуя инструкциям в документации по [аутентификации](../security/authentication.md), [сбросу пароля](../security/password-reset.md) и [подтверждению электронной почты](../security/email-verification.md).
{% endhint %}

### Что такое Fortify? <a id="what-is-fortify"></a>

As mentioned previously, Laravel Fortify is a frontend agnostic authentication backend implementation for Laravel. Fortify registers the routes and controllers needed to implement all of Laravel's authentication features, including login, registration, password reset, email verification, and more.

**Вы не обязаны использовать Fortify для аутентификации в Laravel.** Вы всегда можете вручную воспользоваться сервисами Laravel по аутентификации, следуя инструкциям в документации по [аутентификации](../security/authentication.md), [сбросу пароля](../security/password-reset.md) и [подтверждению электронной почты](../security/email-verification.md).

Если вы новичок в Laravel, возможно, вы захотите изучить стартовый комплект приложения Laravel Breeze, прежде чем пытаться использовать Laravel Fortify. [Laravel Breeze](../getting-started/starter-kits.md) обеспечивает каркас аутентификации для приложения, который включает пользовательский интерфейс, построенный с помощью [Tailwind CSS](https://tailwindcss.com/). В отличие от Fortify, Breeze публикует свои маршруты и контроллеры непосредственно в вашем приложении. Это позволяет вам изучать и чувствовать себя комфортно с функциями аутентификации Laravel, прежде чем позволить Laravel Fortify реализовать эти функции для вас.

Laravel Fortify в основе своей берет маршруты и контроллеры Laravel Breeze и предлагает их в виде пакета, который не включает в себя пользовательский интерфейс. Это позволяет вам все еще быстро выстраивать внутреннюю реализацию уровня аутентификации вашего приложения, не привязываясь к какому-либо конкретному фронтенду.

### Когда нужно использовать Fortify?

Вы можете задаться вопросом, когда уместно использовать Laravel Fortify. Во-первых, если вы используете один из [стартовых наборов](../getting-started/starter-kits.md) Laravel, вам не нужно устанавливать Laravel Fortify, так как все стартовые наборы Laravel уже обеспечивают полную аутентификацию.

Если вы не используете стартовый комплект приложения и вашему приложению нужны функции аутентификации, у вас есть два варианта: вручную реализовать функции аутентификации приложения или использовать Laravel Fortify для обеспечения бэкэнда реализации этих функций.

Если вы выберете установку Fortify, ваш пользовательский интерфейс будет делать запросы на маршруты аутентификации Fortify для того, чтобы аутентифицировать и зарегистрировать пользователей.

Если вы выберете ручное взаимодействие со службами аутентификации Laravel вместо использования Fortify, вы можете сделать это, следуя документации, доступной в документации по [аутентификации](../security/authentication.md), [сбросу пароля](../security/password-reset.md) и [проверке электронной почты](../security/email-verification.md).

#### Laravel Fortify & Laravel Sanctum

Некоторые разработчики путаются в разнице между [Laravel Sanctum](sanctum.md) и Laravel Fortify. Поскольку эти два пакета решают две разные, но связанные между собой проблемы, Laravel Fortify и Laravel Sanctum не являются взаимоисключающими или конкурирующими пакетами.

Laravel Sanctum занимается только управлением токенами API и аутентификацией существующих пользователей с помощью сессионных cookie-файлов или токенов. Sanctum не предоставляет никаких маршрутов для регистрации пользователей, сброса пароля и т.д.

Если вы пытаетесь вручную построить уровень аутентификации для приложения, которое предлагает API или служит бэкэндом для одностраничного приложения, вполне возможно, что вы будете использовать как Laravel Fortify \(для регистрации пользователей, сброса пароля и т.д.\), так и Laravel Sanctum \(управление токенами API, сеансовая аутентификация\).

## Установка <a id="installation"></a>

Для начала установите Fortify с помощью менеджера пакетов Composer:

```bash
composer require laravel/fortify
```

Далее опубликуйте ресурсы Fortify с помощью команды `vendor:publish`:

```bash
php artisan vendor:publish --provider="Laravel\Fortify\FortifyServiceProvider"
```

Эта команда опубликует действия Fortify в вашем каталоге `app/Actions`, который будет создан, если его не существует. Кроме того, будут опубликованы конфигурационный файл Fortify и миграции.

Далее, вы должны мигрировать вашу базу данных:

```bash
php artisan migrate
```

### Сервис-провайдер Fortify

Команда `vendor:publish`, рассмотренная выше, также опубликует класс `App\Providers\FortifyServiceProvider`. Необходимо убедиться, что этот класс зарегистрирован в массиве провайдеров конфигурационного файла `config/app.php` вашего приложения.

Сервис-провайдер Fortify регистрирует опубликованные действия Fortify и дает указание Fortify использовать их, когда соответствующие задания выполняются Fortify.

### Функции Fortify

Файл конфигурации `fortify` содержит массив настроек функций `features`. Этот массив определяет, какие внутренние маршруты \(`routes`\) / функции \(`features`\) Fortify будут выставлены по умолчанию. Если вы не используете Fortify в сочетании с [Laravel Jetstream](https://jetstream.laravel.com/), мы рекомендуем включить только следующие функции, которые являются основными функциями аутентификации, предоставляемыми большинством приложений Laravel:

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

### Отключение представлений

По умолчанию Fortify определяет маршруты, которые предназначены для возврата представлений, таких как экран входа в систему или экран регистрации. Однако, если вы создаете одностраничное приложение на JavaScript, вам могут не понадобиться эти маршруты. По этой причине вы можете полностью их отключить, установив значение конфигурации представлений \(`views`\) в конфигурационном файле `config/fortify.php` вашего приложения в `false`:

```php
'views' => false,
```

#### Отключение представлений и сброс пароля

Если вы решили отключить представления Fortify, и вы будете реализовывать функции сброса пароля для вашего приложения, вы все равно должны определить маршрут с именем `password.reset`, который отвечает за отображение представления "сброса пароля". Это необходимо, потому что уведомление Laravel `Illuminate\Auth\Notifications\ResetPassword` сгенерирует URL сброса пароля через маршрут с именем `password.reset`.

## Аутентификация

Чтобы начать, нам нужно проинструктировать Fortify, как вернуть наш вид "входа". Помните, Fortify — это библиотека аутентификации без фронта. Если вы хотите, реализованную функция аутентификации Laravel во фронтэнде, то должны использовать [стартовый набор приложения](../getting-started/starter-kits.md).

Вся логика отображения представлений аутентификации может быть настроена с помощью соответствующих методов, доступных в классе `Laravel\Fortify\Fortify`. Как правило, этот метод следует вызывать из метода `boot` класса `App\Providers\FortifyServiceProvider`. Fortify позаботится об определении маршрута `/login`, который возвращает это представление:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::loginView(fn () => view('auth.login'));

    // ...
}
```

Ваш шаблон входа должен включать форму, которая делает POST-запрос на `/login`. Конечная точка `/login` ожидает _адрес электронной почты / имя пользователя_ и `password` \(пароль\). Имя поля _электронной почты / имя пользователя_ должно совпадать со значением `username` в конфигурационном файле `config/fortify.php`. Кроме того, может быть предоставлено булевое поле `remember` \(запомнить\), чтобы указать, что пользователь хотел бы использовать функциональность "запомнить меня", предоставляемую Laravel.

Если попытка входа прошла успешно, Fortify перенаправит вас к URI, настроенному через опцию `home` в файле конфигурации `fortify` вашего приложения. Если запрос на вход был XHR-запросом, то будет возвращен ответ 200 HTTP.

Если запрос не увенчался успехом, пользователь будет перенаправлен обратно на экран входа в систему, а ошибки валидации будут доступны через [общедоступную переменную](../validation.md#displaying-the-validation-errors) `$errors` в Blade шаблоне. Или, в случае XHR запроса, ошибки валидации будут возвращены с ответом 422 HTTP.

### Настройка аутентификации

Fortify автоматически получит и аутентифицирует пользователя, основываясь на предоставленных учетных данных и защите аутентификации, настроенной для приложения. Однако иногда вам может понадобиться полная настройка того, как аутентифицируются учетные данные и как пользователи получают их. К счастью, Fortify позволяет вам легко выполнить это с помощью метода `Fortify::authenticateUsing`.

Этот метод принимает функцию обратного вызова, которая получает входящий HTTP-запрос. Функция отвечает за проверку учетных данных, прикрепленных к запросу, и возврат связанного с ним экземпляра пользователя. Если учетные данные недействительны или ни один пользователь не может быть найден, то должен быть возвращен `null` или `false`. Обычно этот метод вызывается из метода boot вашего `FortifyServiceProvider`:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::authenticateUsing(function (Request $request) {
        $user = User::where('email', $request->email)->first();

        if ($user &&
            Hash::check($request->password, $user->password)) {
            return $user;
        }
    });

    // ...
}
```

#### Защитники аутентификации

Вы можете настроить защитников аутентификации, используемых Fortify в файле конфигурации `fortify`. Однако, вы должны убедиться, что настроенный защитник является реализацией `Illuminate\Contracts\Auth\StatefulGuard`. Если вы пытаетесь использовать Laravel Fortify для аутентификации SPA, вы должны использовать стандартный защитник `web` в сочетании с [Laravel Sanctum](sanctum.md).

## Двухфакторная аутентификация

When Fortify's two factor authentication feature is enabled, the user is required to input a six digit numeric token during the authentication process. This token is generated using a time-based one-time password \(TOTP\) that can be retrieved from any TOTP compatible mobile authentication application such as Google Authenticator.

Before getting started, you should first ensure that your application's `App\Models\User` model uses the `Laravel\Fortify\TwoFactorAuthenticatable` trait:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use Notifiable, TwoFactorAuthenticatable;
}
```

Next, you should build a screen within your application where users can manage their two factor authentication settings. This screen should allow the user to enable and disable two factor authentication, as well as regenerate their two factor authentication recovery codes.

По умолчанию массив `feature` файла конфигурации `fortify` инструктирует Fortify о двухфакторной аутентификации, чтобы требовать подтверждения пароля перед модификацией. Поэтому перед продолжением работы в приложении должна быть реализована функция [подтверждения пароля](fortify.md#password-confirmation) Fortify.

### Enabling Two Factor Authentication

To enable two factor authentication, your application should make a POST request to the `/user/two-factor-authentication` endpoint defined by Fortify. If the request is successful, the user will be redirected back to the previous URL and the `status` session variable will be set to `two-factor-authentication-enabled`. You may detect this `status` session variable within your templates to display the appropriate success message. If the request was an XHR request, `200` HTTP response will be returned:

```markup
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm text-green-600">
        Two factor authentication has been enabled.
    </div>
@endif
```

Next, you should display the two factor authentication QR code for the user to scan into their authenticator application. If you are using Blade to render your application's frontend, you may retrieve the QR code SVG using the `twoFactorQrCodeSvg` method available on the user instance:

```php
$request->user()->twoFactorQrCodeSvg();
```

If you are building a JavaScript powered frontend, you may make an XHR GET request to the `/user/two-factor-qr-code` endpoint to retrieve the user's two factor authentication QR code. This endpoint will return a JSON object containing an `svg` key.

**Displaying The Recovery Codes**

You should also display the user's two factor recovery codes. These recovery codes allow the user to authenticate if they lose access to their mobile device. If you are using Blade to render your application's frontend, you may access the recovery codes via the authenticated user instance:

```php
(array) $request->user()->two_factor_recovery_codes
```

If you are building a JavaScript powered frontend, you may make an XHR GET request to the `/user/two-factor-recovery-codes` endpoint. This endpoint will return a JSON array containing the user's recovery codes.

To regenerate the user's recovery codes, your application should make a POST request to the `/user/two-factor-recovery-codes` endpoint.

### Authenticating With Two Factor Authentication

During the authentication process, Fortify will automatically redirect the user to your application's two factor authentication challenge screen. However, if your application is making an XHR login request, the JSON response returned after a successful authentication attempt will contain a JSON object that has a `two_factor` boolean property. You should inspect this value to know whether you should redirect to your application's two factor authentication challenge screen.

To begin implementing two factor authentication functionality, we need to instruct Fortify how to return our two factor authentication challenge view. All of Fortify's authentication view rendering logic may be customized using the appropriate methods available via the `Laravel\Fortify\Fortify` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\FortifyServiceProvider` class:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::twoFactorChallengeView(function () {
        return view('auth.two-factor-challenge');
    });

    // ...
}
```

Fortify will take care of defining the `/two-factor-challenge` route that returns this view. Your `two-factor-challenge` template should include a form that makes a POST request to the `/two-factor-challenge` endpoint. The `/two-factor-challenge` action expects a `code` field that contains a valid TOTP token or a `recovery_code` field that contains one of the user's recovery codes.

If the login attempt is successful, Fortify will redirect the user to the URI configured via the `home` configuration option within your application's `fortify` configuration file. If the login request was an XHR request, a 204 HTTP response will be returned.

Если запрос не увенчался успехом, пользователь будет перенаправлен обратно на экран входа в систему, а ошибки валидации будут доступны через общедоступную [шаблонную переменную](../validation.md#displaying-the-validation-errors) `$errors`. Или, в случае XHR запроса, ошибки валидации будут возвращены с ответом 422 HTTP.

### Disabling Two Factor Authentication

Чтобы отключить двухфакторную аутентификацию, ваше приложение должно сделать запрос DELETE к конечной точке `/user/two-factor-authentication`. Помните, что конечные точки двухфакторной аутентификации Fortify требуют [подтверждения пароля](fortify.md#password-confirmation) перед вызовом.

## Регистрация

Чтобы начать реализацию функционала регистрации нашего приложения, нам необходимо проинструктировать Fortify о том, как вернуть наш "регистрационный" вид. Помните, Fortify - это библиотека аутентификации без фронта. Если вы хотите, чтобы во фронтэнде была реализована функция аутентификации Laravel, используйте [стартовый комплект приложения](../getting-started/starter-kits.md).

Вся логика рендеринга представления Fortify может быть настроена с помощью соответствующих методов, доступных в классе `Laravel\Fortify\Fortify`. Как правило, этот метод следует вызывать из метода `boot` класса `App\Providers\FortifyServiceProvider`:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::registerView(fn () => view('auth.register'));

    // ...
}
```

Fortify позаботится об определении маршрута `/register`, который возвращает этот вид. Ваш шаблон регистрации должен включать форму, которая делает POST-запрос к конечной точке `/register`, определенной Fortify.

Конечная точка `/register` ожидает поля `name`, _адрес электронной почты / имя пользователя_, `password` и `password_confirmation`. Имя поля _электронной почты / имя пользователя_ должно соответствовать значению `username`, определенному в файле конфигурации `fortify` вашего приложения.

Если попытка регистрации прошла успешно, Fortify перенаправит пользователя к URI, сконфигурированному через опцию `home` в файле конфигурации `fortify`. Если запрос на вход был XHR-запросом, то будет возвращен ответ 200 HTTP.

Если запрос не увенчался успехом, пользователь будет перенаправлен обратно на экран регистрации, а ошибки валидации будут доступны через общедоступную [переменную](../validation.md#displaying-the-validation-errors) `$errors` в Blade шаблоне. Или, в случае XHR запроса, ошибки валидации будут возвращены с ответом 422 HTTP.

### Настройка регистрации

Процесс проверки и создания пользователя можно настроить, изменив действие `App\Actions\Fortify\CreateNewUser`, которое было сгенерировано при установке Laravel Fortify.

## Сброс пароля

### Запрос ссылки на сброс пароля

Чтобы начать реализацию функции сброса пароля в нашем приложении, нам необходимо проинструктировать Fortify о том, как вернуть наш вид "забытого пароля". Помните, Fortify - это библиотека аутентификации без фронта. Если вы хотите, чтобы во фронтэнде была реализована функция аутентификации Laravel, используйте [стартовый комплект приложения](../getting-started/starter-kits.md).

Вся логика рендеринга представления Fortify может быть настроена с помощью соответствующих методов, доступных в классе `Laravel\Fortify\Fortify`. Как правило, этот метод следует вызывать из метода `boot` класса `App\Providers\FortifyServiceProvider`:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::requestPasswordResetLinkView(function () {
        return view('auth.forgot-password');
    });

    // ...
}
```

Fortify позаботится об определении конечной точки `/forgot-password`, которая возвращает это представление. Ваш шаблон забытого пароля должен включать форму, которая делает POST-запрос к конечной точке `/forgot-password`.

Конечная точка `/forgot-password` ожидает строковое поле `email`. Имя этого поля / столбца базы данных должно совпадать со значением опции `email` в файле конфигурации `fortify` вашего приложения.

#### Обработка ответа на запрос ссылки сброса пароля

Если запрос ссылки на сброс пароля был успешным, Fortify перенаправит пользователя обратно на конечную точку `/forgot-password` и отправит электронное письмо пользователю с защищенной ссылкой, которую он может использовать для сброса своего пароля. Если запрос был XHR, то будет возвращен ответ 200 HTTP.

После перенаправления обратно в конечную точку `/forgot-password` после успешного запроса, переменная сессии `status` может быть использована для отображения статуса попытки запроса ссылки на сброс пароля. Значение этой переменной сессии будет соответствовать одной из строк перевода, определенных [в файле языка](../localization.md) `passwords` вашего приложения:

```markup
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

Если запрос не увенчался успехом, пользователь будет перенаправлен обратно на экран запроса ссылки сброса пароля, а ошибки проверки будут доступны через общедоступную [переменную Blade шаблона](../validation.md#displaying-the-validation-errors) `$errors`. Или, в случае XHR-запроса, ошибки валидации будут возвращены с ответом 422 HTTP.

### Сброс пароля

Чтобы закончить реализацию функции сброса пароля нашего приложения, мы должны инструктировать Fortify, как вернуть представление "сброс пароля".

Вся логика рендеринга представления Fortify может быть настроена с помощью соответствующих методов, доступных в классе `Laravel\Fortify\Fortify`. Как правило, этот метод следует вызывать из метода `boot` класса `App\Providers\FortifyServiceProvider`:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::resetPasswordView(function ($request) {
        return view('auth.reset-password', ['request' => $request]);
    });

    // ...
}
```

Fortify позаботится об определении маршрута для отображения этого вида. Ваш шаблон сброса пароля должен включать форму, которая делает POST-запрос на `/reset-password`.

Конечная точка `/reset-password` ожидает поле _электронной почты_, поле `password`, поле `password_confirmation` и скрытое поле с именем `token`, содержащее значение `request()->route('token')`. Имя поля _"email"_ / столбца базы данных должно совпадать со значением опции `email`, определенным в конфигурационном файле `fortify` вашего приложения.

#### Обработка ответа на сброс пароля

Если запрос на сброс пароля был успешным, Fortify переадресовывает обратно на маршрут `/login`, чтобы пользователь мог войти в систему с новым паролем. Кроме того, будет установлена переменная сессии `status`, чтобы вы могли отобразить успешный статус сброса на экране входа в систему:

```php
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

Если запрос был XHR-запросом, то будет возвращен ответ 200 HTTP.

Если запрос не увенчался успехом, пользователь будет перенаправлен обратно на экран сброса пароля, а ошибки валидации будут доступны через общедоступную [переменную шаблона](../validation.md#displaying-the-validation-errors) `$errors`. Или, в случае XHR-запроса, ошибки валидации будут возвращены с ответом 422 HTTP.

### Настройка сброса пароля

Процесс сброса пароля можно настроить, изменив действие `App\Actions\ResetUserPassword`, которое было сгенерировано при установке Laravel Fortify.

## Подтверждение адреса электронной почты

После регистрации пользователи могут подтвердить свой адрес электронной почты перед тем, как продолжить доступ к вашему приложению. Для начала убедитесь, что функция `emailVerification` включена в массиве функций файла конфигурации `fortify`. Далее вы должны убедиться, что ваш класс `App\Models\User` реализует интерфейс `Illuminate\Contracts\Auth\MustVerifyEmail`.

После завершения этих двух этапов настройки новые зарегистрированные пользователи получат электронное сообщение с просьбой подтвердить свой электронный адрес. Однако, нам нужно сообщить Fortify о том, как отобразить экран подтверждения электронной почты, который информирует пользователя о том, что ему необходимо перейти по ссылке в электронном сообщении.

Вся логика рендеринга представления Fortify может быть настроена с помощью соответствующих методов, доступных в классе `Laravel\Fortify\Fortify`. Как правило, этот метод следует вызывать из метода `boot` класса `App\Providers\FortifyServiceProvider`:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::verifyEmailView(fn () => view('auth.verify-email'));

    // ...
}
```

Fortify позаботится об определении маршрута, который отображает этот вид, когда пользователь перенаправляется на конечную точку `/email/verify` встроенным посредником `verified`.

Your `verify-email` template should include an informational message instructing the user to click the email verification link that was sent to their email address.

**Resending Email Verification Links**

If you wish, you may add a button to your application's `verify-email` template that triggers a POST request to the `/email/verification-notification` endpoint. When this endpoint receives a request, a new verification email link will be emailed to the user, allowing the user to get a new verification link if the previous one was accidentally deleted or lost.

If the request to resend the verification link email was successful, Fortify will redirect the user back to the `/email/verify` endpoint with a `status` session variable, allowing you to display an informational message to the user informing them the operation was successful. If the request was an XHR request, a 202 HTTP response will be returned:

```markup
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        A new email verification link has been emailed to you!
    </div>
@endif
```

### Защита маршрутов

Чтобы указать, что маршрут или группа маршрутов требуют, чтобы пользователь подтвердил свой адрес электронной почты, необходимо прикрепить к маршруту посредника `verified`. Этот посредник зарегистрирован в классе `App\Http\Kernel`:

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

## Подтверждение пароля <a id="password-confirmation"></a>

While building your application, you may occasionally have actions that should require the user to confirm their password before the action is performed. Typically, these routes are protected by Laravel's built-in `password.confirm` middleware.

Чтобы приступить к реализации функции подтверждения пароля, нам необходимо проинструктировать Fortify о том, как вернуть нашему приложению вид "подтверждения пароля". Помните, Fortify — это библиотека аутентификации без фронта. Если вы хотите, чтобы во фронтэнде была реализована функция аутентификации Laravel, вы должны использовать [стартовый комплект приложения](../getting-started/starter-kits.md).

All of Fortify's view rendering logic may be customized using the appropriate methods available via the `Laravel\Fortify\Fortify` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\FortifyServiceProvider` class:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Fortify::confirmPasswordView(function () {
        return view('auth.confirm-password');
    });

    // ...
}
```

Fortify will take care of defining the `/user/confirm-password` endpoint that returns this view. Your `confirm-password` template should include a form that makes a POST request to the `/user/confirm-password` endpoint. The `/user/confirm-password` endpoint expects a `password` field that contains the user's current password.

If the password matches the user's current password, Fortify will redirect the user to the route they were attempting to access. If the request was an XHR request, a 201 HTTP response will be returned.

If the request was not successful, the user will be redirected back to the confirm password screen and the validation errors will be available to you via the shared `$errors` Blade template variable. Or, in the case of an XHR request, the validation errors will be returned with a 422 HTTP response.

