# Почта

## Вступление

Отправка электронной почты не обязательно должна быть сложной. Laravel предоставляет простой и понятный API для электронной почты, работающий на базе популярной библиотеки [SwiftMailer](https://swiftmailer.symfony.com/). Laravel и SwiftMailer предоставляют драйверы для отправки электронной почты по SMTP, Mailgun, Postmark, Amazon SES и `sendmail`, позволяя вам легко организовать отправку почты через локальный или облачный сервис по вашему выбору.

### Конфигурация

Службы электронной почты Laravel могут быть сконфигурированы через конфигурационный файл вашего `config/mail.php`. Каждый почтовый клиент, сконфигурированный в этом файле, может иметь свою уникальную конфигурацию и даже свой собственный уникальный "транспорт", позволяющий вашему приложению использовать различные почтовые сервисы для отправки определенных сообщений электронной почты. Например, ваше приложение может использовать Postmark для отправки транзакционных сообщений электронной почты, в то время как Amazon SES используется для отправки массовых сообщений электронной почты.

В файле конфигурации `mail` вы найдете массив настроек почтовых серверов `mailers`. Этот массив содержит примеры конфигов для каждого из основных почтовых драйверов/транспортов, поддерживаемых Laravel, в то время как значение конфигурации `default` определяет, какой почтовый сервер будет использоваться по умолчанию, когда вашему приложению необходимо отправить сообщение по электронной почте.

### Требования для драйверов и транспортов

Драйверы, основанные на API, такие как Mailgun и Postmark, зачастую проще и быстрее, чем отправка почты через SMTP-серверы. Когда это возможно, мы рекомендуем использовать один из этих драйверов. Все драйверы, основанные на API, требуют наличия HTTP-библиотеки Guzzle, которая может быть установлена через менеджер пакетов Composer:

```text
composer require guzzlehttp/guzzle
```

#### **Драйвер Mailgun**

Чтобы использовать драйвер Mailgun, сначала установите HTTP-библиотеку Guzzle. Затем установите опцию `default` в конфигурационном файле `config/mail.php` в `mailgun`. Далее убедитесь, что ваш конфигурационный файл `config/services.php` содержит следующие опции:

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
],
```

Если вы не используете [регион US Mailgun](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions), вы можете определить конечную точку вашего региона в файле конфигурации `services`:

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
],
```

#### **Драйвер Postmark**

Чтобы использовать Postmark драйвер, установите Postmark's SwiftMailer транспорт через Composer:

```text
composer require wildbit/swiftmailer-postmark
```

Далее установите HTTP-библиотеку Guzzle и установите параметр `default` в файле конфигурации `config/mail.php` в значение `postmark`. Наконец, убедитесь, что ваш конфигурационный файл `config/services.php` содержит следующие опции:

```php
'postmark' => [
    'token' => env('POSTMARK_TOKEN'),
],
```

Если вы хотите указать поток сообщений Postmark, который должен использоваться данным мейлером, то можете добавить опцию конфигурации `message_stream_id` в массив настроек мейлера. Этот массив можно найти в конфигурационном файле вашего приложения `config/mail.php`:

```php
'postmark' => [
    'transport' => 'postmark',
    'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
],
```

Таким образом, вы также сможете настроить несколько почтовых отправителей с различными потоками сообщений.

#### **Драйвер SES**

Для использования драйвера Amazon SES необходимо сначала установить Amazon AWS SDK для PHP. Вы можете установить эту библиотеку через менеджер пакетов Composer:

```text
composer require aws/aws-sdk-php
```

Далее установите опцию `default` в вашем конфигурационном файле `config/mail.php` в `ses` и убедитесь, что ваш конфигурационный файл `config/services.php` содержит следующие опции:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
```

Если вы хотите определить [дополнительные опции](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-email-2010-12-01.html#sendrawemail), которые Laravel должна передать методу `SendRawEmail` в AWS SDK при отправке электронной почты, вы можете определить массив `options` в вашей `ses` конфигурации:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'options' => [
        'ConfigurationSetName' => 'MyConfigurationSet',
        'Tags' => [
            ['Name' => 'foo', 'Value' => 'bar'],
        ],
    ],
],
```

## Генерация классов почтовых отправлений

При создании приложений Laravel, каждый тип почтовых отправлений представлен в виде класса "mailable". Эти классы хранятся в директории `app/Mail`. Не волнуйтесь, если вы не видите этот каталог в вашем приложении, так как он будет сгенерирован автоматически при создании первого класса письма, отправляемого по почте, с помощью команды `make:mail`:

```text
php artisan make:mail OrderShipped
```

## Написание почтовых классов

После того, как вы сгенерировали почтовый класс, откройте его, чтобы посмотреть его содержимое. Во-первых, обратите внимание, что вся настройка класса mailable производится в методе `build`. В этом методе можно вызывать различные методы, например, `from`, `subject`, `view` и `attach`, чтобы настроить представление и доставку письма.

### Настройка отправителя

#### **Используя метод** `from`

Сначала давайте изучим настройку отправителя электронной почты. Или, другими словами, кто будет указан в поле "от" электронного письма. Есть два способа настроить отправителя. Во-первых, вы можете использовать метод `from` в методе `build` вашего класса mailable:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->from('example@example.com')
                ->view('emails.orders.shipped');
}
```

#### **Используя глобальную настройку**

С другой стороны, если ваше приложение использует один и тот же адрес "от" для всех своих писем, то вызов метода `from` в каждом генерируемом вами почтовом классе может стать обременительным. Вместо этого, вы можете указать глобальный адрес "from" в конфигурационном файле `config/mail.php`. Этот адрес будет использоваться, если в почтовом классе не указан другой адрес "from":

```php
'from' => ['address' => 'example@example.com', 'name' => 'App Name'],
```

Кроме того, вы можете определить глобальный адрес "reply\_to" в конфиге `config/mail.php`:

```php
'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],
```

### Настройка отображения

В методе `build` класса mailable можно использовать метод `view`, чтобы указать, какой шаблон следует использовать при отрисовке содержимого письма. Так как каждое письмо обычно использует шаблон Blade для отрисовки своего содержимого, у вас есть полная мощь и удобство шаблонизатора Blade при построении HTML вашего письма:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped');
}
```

{% hint style="info" %}
Предлагаем вам создать каталог `resources/views/emails` для размещения всех ваших почтовых шаблонов; однако, вы можете разместить их где угодно в вашем каталоге `resources/views`.
{% endhint %}

#### **Письма в виде простого текста**

Если вы хотите определить текстовую версию своего электронного письма, вы можете использовать метод `text`. Как и метод `view`, метод `text` принимает имя шаблона, которое будет использоваться для отображения содержимого письма. Вы можете определить как HTML, так и обычную текстовую версию вашего сообщения:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped')
                ->text('emails.orders.shipped_plain');
}
```

### Данные представления

#### **Через публичные свойства**

Как правило, вы захотите передать некоторые данные для представления, которые можно использовать при рендеринге HTML сообщения электронной почты. Есть два способа сделать данные доступными для просмотра. Во-первых, любое публичное свойство, определенное в вашем классе почтовой службы, будет автоматически сделано доступным в представлении. Так, например, вы можете передать данные в конструктор вашего класса mailable и установить эти данные в публичные свойства, определенные в этом классе:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * The order instance.
     *
     * @var \App\Models\Order
     */
    public $order;

    /**
     * Create a new message instance.
     *
     * @param  \App\Models\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }
}
```

Как только данные будут установлены в публичное свойство, они автоматически будут доступны в вашем представлении, так что вы можете получить доступ к ним, как к любым другим данным в ваших шаблонах Blade:

```markup
<div>
    Price: {{ $order->price }}
</div>
```

#### **Через метод `with`:**

Если вы хотите настроить формат данных вашего электронного письма до того, как оно будет отправлено в шаблон, вы можете вручную передать свои данные для представления с помощью метода `with`. Обычно вы все равно передаете данные через конструктор класса mailable; однако, вы должны установить эти данные в _защищенные_ или _приватные_ свойства, чтобы данные не становились автоматически доступными для шаблона. Затем, при вызове метода `with`, передайте массив данных, который вы хотите сделать доступным для шаблона:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * The order instance.
     *
     * @var \App\Models\Order
     */
    protected $order;

    /**
     * Create a new message instance.
     *
     * @param  \App\Models\Order $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->with([
                        'orderName' => $this->order->name,
                        'orderPrice' => $this->order->price,
                    ]);
    }
}
```

Как только данные будут переданы в метод `with`, они автоматически будут доступны в вашем представлении, так что вы можете получить доступ к ним, как к любым другим данным в ваших шаблонах Blade:

```markup
<div>
    Price: {{ $orderPrice }}
</div>
```

### Вложения

Для прикрепления вложений к письму используйте метод `attach` в методе `build` класса mailable'. Метод `attach` принимает в качестве первого аргумента полный путь к файлу:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped')
                ->attach('/path/to/file');
}
```

При прикреплении файлов к сообщению, вы также можете указать отображаемое имя и/или MIME-тип, передавая массив в качестве второго аргумента в метод `attach`:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped')
                ->attach('/path/to/file', [
                    'as' => 'name.pdf',
                    'mime' => 'application/pdf',
                ]);
}
```

#### Прикрепление файла с диска

Если вы сохранили файл на одном из дисков вашей [файловой системы](filesystem.md), то можете прикрепить его к письму, используя метод `attachFromStorage`:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
   return $this->view('emails.orders.shipped')
               ->attachFromStorage('/path/to/file');
}
```

При необходимости можно указать имя прикрепленного файла и дополнительные опции, используя второй и третий аргументы метода `attachFromStorage`:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
   return $this->view('emails.orders.shipped')
               ->attachFromStorage('/path/to/file', 'name.pdf', [
                   'mime' => 'application/pdf'
               ]);
}
```

Метод `attachFromStorageDisk` может быть использован, если необходимо указать диск хранения, отличный от диска по умолчанию:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
   return $this->view('emails.orders.shipped')
               ->attachFromStorageDisk('s3', '/path/to/file');
}
```

#### **Raw Data Attachments**

Метод `attachData` может быть использован для прикрепления необработанной строки байтов в качестве вложения. Например, вы можете использовать этот метод, если вы сгенерировали PDF в памяти и хотите прикрепить его к письму, не записывая на диск. Метод `attachData` принимает байты исходных данных в качестве первого аргумента, имя файла в качестве второго аргумента и массив опций в качестве третьего аргумента:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped')
                ->attachData($this->pdf, 'name.pdf', [
                    'mime' => 'application/pdf',
                ]);
}
```

### Встроенные изображения

Встраивание изображений в электронную почту обычно трудоемко, однако Laravel предоставляет удобный способ прикрепления изображений к электронным письмам. Чтобы вставить изображение, используйте метод `embed` в переменной `$message` в вашем шаблоне электронной почты. Laravel автоматически делает переменную `$message` доступной для всех ваших почтовых шаблонов, так что вам не нужно беспокоиться о том, чтобы передать ее вручную:

```markup
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

{% hint style="danger" %}
Переменная `$message` недоступна в шаблонах обычных текстовых сообщений, так как в обычных текстовых сообщениях не используются встроенные вложения.
{% endhint %}

#### **Embedding Raw Data Attachments**

Если у вас уже есть необработанная строка данных изображения, которую вы хотите вставить в почтовый шаблон, вы можете вызвать метод `embedData` в переменной `$message`. При вызове метода `embedData` вам нужно будет указать имя файла, который должен быть присвоен встроенному изображению:

```markup
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

### Настройка сообщения SwiftMailer

Метод `withSwiftMessage` базового класса `Mailable` позволяет зарегистрировать замыкание, которое будет вызвано экземпляром сообщения SwiftMailer перед отправкой сообщения. Это дает возможность полностью настроить сообщение до того, как оно будет доставлено:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    $this->view('emails.orders.shipped');

    $this->withSwiftMessage(function ($message) {
        $message->getHeaders()->addTextHeader(
            'Custom-Header', 'Header Value'
        );
    });
}
```

## Markdown письма

Почтовые сообщения Markdown позволяют вам воспользоваться предварительно созданными шаблонами и компонентами [почтовых уведомлений](notifications.md#mail-notifications) в ваших почтовых файлах. Так как сообщения написаны в Markdown, Laravel способен отображать красивые, отзывчивые HTML шаблоны для писем, а также автоматически генерировать обычный текстовый контрагент.

### Генерация Markdown писем

Для генерации письма с соответствующим шаблоном Markdown можно использовать опцию `--markdown` команды `make:mail`:

```text
php artisan make:mail OrderShipped --markdown=emails.orders.shipped
```

Затем, при настройке письма в его методе `build`, вызовите метод `markdown`, а не метод `view`. Метод `markdown` принимает имя шаблона Markdown и необязательный массив данных, который будет доступен шаблону:

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->from('example@example.com')
                ->markdown('emails.orders.shipped', [
                    'url' => $this->orderUrl,
                ]);
}
```

### Написание Markdown сообщений

Письма Markdown используют комбинацию компонентов Blade и синтаксис Markdown, что позволяет легко создавать почтовые сообщения, используя предварительно встроенные компоненты почтового пользовательского интерфейса Laravel:

```markup
@component('mail::message')
# Order Shipped

Your order has been shipped!

@component('mail::button', ['url' => $url])
View Order
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

{% hint style="info" %}
Не используйте лишние отступы при записи писем Markdown. В соответствии со стандартами Markdown парсеры Markdown будут отображать содержимое с отступами в виде блоков кода.
{% endhint %}

#### **Button Component**

Компонент `button` создает центрированную кнопку-ссылку. Компонент принимает два аргумента, `url` и необязательный `color`. Поддерживаемые цвета — `primary`, `success` и `error`. Вы можете добавить в сообщение столько компонентов кнопок, сколько пожелаете:

```markup
@component('mail::button', ['url' => $url, 'color' => 'success'])
View Order
@endcomponent
```

#### **Panel Component**

Компонент панели отображает заданный блок текста на панели, цвет фона которой немного отличается от цвета остальной части сообщения. Это позволяет привлечь внимание к заданному блоку текста:

```markup
@component('mail::panel')
This is the panel content.
@endcomponent
```

#### **Table Component**

Компонент таблицы позволяет преобразовать таблицу Markdown в HTML-таблицу. Компонент принимает Markdown table в качестве содержимого. Выравнивание столбцов таблицы поддерживается синтаксисом выравнивания по умолчанию:

```markup
@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
```

### Настройка компонентов

Вы можете экспортировать все компоненты почты Markdown в ваше собственное приложение для настройки. Для экспорта компонентов воспользуйтесь командой `vendor:publish`, чтобы опубликовать тег ресурсов `laravel-mail`:

```text
php artisan vendor:publish --tag=laravel-mail
```

Эта команда опубликует почтовые компоненты Markdown в каталоге `resources/views/vendor/mail`. Каталог `mail` будет содержать подкаталоги `html` и `text`, каждый из которых содержит соответствующие представления каждого доступного компонента. Вы можете настроить эти компоненты как угодно.

#### **Настройка CSS**

После экспорта компонентов каталог `resources/views/vendor/mail/html/themes` будет содержать файл `default.css`. Вы можете настроить CSS в этом файле, и ваши стили будут автоматически преобразованы в inline CSS в HTML представлениях ваших почтовых сообщений Markdown.

Если вы хотите создать совершенно новую тему для компонентов Laravel's Markdown, то можете поместить CSS-файл в каталог `html/themes`. После определения имени и сохранения CSS-файла обновите параметр `theme` конфигурационного файла `config/mail.php` приложения, чтобы он совпал с именем вашей новой темы.

Чтобы настроить тему для отдельного почтового класса, вы можете установить свойство `$theme` класса mailable в соответствие с именем темы, которая будет использоваться при отправке этого почтового класса.

## Отправка почты

Чтобы отправить сообщение, используйте метод `to` на [фасаде](facades.md) `Mail`. Метод `to` принимает адрес электронной почты, экземпляр пользователя или коллекцию пользователей. Если вы передаете объект или коллекцию объектов, почтовый агент автоматически использует их свойства `email` и `name` при определении получателей электронной почты, поэтому убедитесь, что эти атрибуты доступны в ваших объектах. После того, как вы указали получателей, можете передать экземпляр своего почтового класса в метод `send`:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Mail\OrderShipped;
use App\Models\Order;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $order = Order::findOrFail($request->order_id);

        // Ship the order...

        Mail::to($request->user())->send(new OrderShipped($order));
    }
}
```

Вы не ограничиваетесь только указанием получателей "кому" при отправке сообщения. Вы можете установить получателей "to", "cc" и "bcc", связав их соответствующие методы вместе:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

#### **Looping Over Recipients**

Occasionally, you may need to send a mailable to a list of recipients by iterating over an array of recipients / email addresses. However, since the `to` method appends email addresses to the mailable's list of recipients, each iteration through the loop will send another email to every previous recipient. Therefore, you should always re-create the mailable instance for each recipient:

```php
foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
    Mail::to($recipient)->send(new OrderShipped($order));
}
```

#### **Sending Mail Via A Specific Mailer**

By default, Laravel will send email using the mailer configured as the `default` mailer in your application's `mail` configuration file. However, you may use the `mailer` method to send a message using a specific mailer configuration:

```php
Mail::mailer('postmark')
        ->to($request->user())
        ->send(new OrderShipped($order));
```

### Queueing Mail

#### **Queueing A Mail Message**

Since sending email messages can negatively impact the response time of your application, many developers choose to queue email messages for background sending. Laravel makes this easy using its built-in [unified queue API](https://laravel.com/docs/8.x/queues). To queue a mail message, use the `queue` method on the `Mail` facade after specifying the message's recipients:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```

This method will automatically take care of pushing a job onto the queue so the message is sent in the background. You will need to [configure your queues](https://laravel.com/docs/8.x/queues) before using this feature.

#### **Delayed Message Queueing**

If you wish to delay the delivery of a queued email message, you may use the `later` method. As its first argument, the `later` method accepts a `DateTime` instance indicating when the message should be sent:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later(now()->addMinutes(10), new OrderShipped($order));
```

#### **Pushing To Specific Queues**

Since all mailable classes generated using the `make:mail` command make use of the `Illuminate\Bus\Queueable` trait, you may call the `onQueue` and `onConnection` methods on any mailable class instance, allowing you to specify the connection and queue name for the message:

```php
$message = (new OrderShipped($order))
                ->onConnection('sqs')
                ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

#### **Queueing By Default**

If you have mailable classes that you want to always be queued, you may implement the `ShouldQueue` contract on the class. Now, even if you call the `send` method when mailing, the mailable will still be queued since it implements the contract:

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    //
}
```

#### **Queued Mailables & Database Transactions**

When queued mailables are dispatched within database transactions, they may be processed by the queue before the database transaction has committed. When this happens, any updates you have made to models or database records during the database transaction may not yet be reflected in the database. In addition, any models or database records created within the transaction may not exist in the database. If your mailable depends on these models, unexpected errors can occur when the job that sends the queued mailable is processed.

If your queue connection's `after_commit` configuration option is set to `false`, you may still indicate that a particular queued mailable should be dispatched after all open database transactions have been committed by defining an `$afterCommit` property on the mailable class:

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    public $afterCommit = true;
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> To learn more about working around these issues, please review the documentation regarding [queued jobs and database transactions](https://laravel.com/docs/8.x/queues#jobs-and-database-transactions).

## Rendering Mailables

Sometimes you may wish to capture the HTML content of a mailable without sending it. To accomplish this, you may call the `render` method of the mailable. This method will return the evaluated HTML content of the mailable as a string:

```php
use App\Mail\InvoicePaid;
use App\Models\Invoice;

$invoice = Invoice::find(1);

return (new InvoicePaid($invoice))->render();
```

### Previewing Mailables In The Browser

When designing a mailable's template, it is convenient to quickly preview the rendered mailable in your browser like a typical Blade template. For this reason, Laravel allows you to return any mailable directly from a route closure or controller. When a mailable is returned, it will be rendered and displayed in the browser, allowing you to quickly preview its design without needing to send it to an actual email address:

```php
Route::get('/mailable', function () {
    $invoice = App\Models\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> [Inline attachments](https://laravel.com/docs/8.x/mail#inline-attachments) will not be rendered when a mailable is previewed in your browser. To preview these mailables, you should send them to an email testing application such as [MailHog](https://github.com/mailhog/MailHog) or [HELO](https://usehelo.com/).

## Localizing Mailables

Laravel allows you to send mailables in a locale other than the request's current locale, and will even remember this locale if the mail is queued.

To accomplish this, the `Mail` facade offers a `locale` method to set the desired language. The application will change into this locale when the mailable's template is being evaluated and then revert back to the previous locale when evaluation is complete:

```php
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);
```

### User Preferred Locales

Sometimes, applications store each user's preferred locale. By implementing the `HasLocalePreference` contract on one or more of your models, you may instruct Laravel to use this stored locale when sending mail:

```php
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    /**
     * Get the user's preferred locale.
     *
     * @return string
     */
    public function preferredLocale()
    {
        return $this->locale;
    }
}
```

Once you have implemented the interface, Laravel will automatically use the preferred locale when sending mailables and notifications to the model. Therefore, there is no need to call the `locale` method when using this interface:

```php
Mail::to($request->user())->send(new OrderShipped($order));
```

## Testing Mailables

Laravel provides several convenient methods for testing that your mailables contain the content that you expect. These methods are: `assertSeeInHtml`, `assertDontSeeInHtml`, `assertSeeInText`, and `assertDontSeeInText`.

As you might expect, the "HTML" assertions assert that the HTML version of your mailable contains a given string, while the "text" assertions assert that the plain-text version of your mailable contains a given string:

```php
use App\Mail\InvoicePaid;
use App\Models\User;

public function test_mailable_content()
{
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertSeeInHtml('Invoice Paid');

    $mailable->assertSeeInText($user->email);
    $mailable->assertSeeInText('Invoice Paid');
}
```

#### **Testing Mailable Sending**

We suggest testing the content of your mailables separately from your tests that assert that a given mailable was "sent" to a specific user. To learn how to test that mailables were sent, check out our documentation on the [Mail fake](https://laravel.com/docs/8.x/mocking#mail-fake).

## Mail & Local Development

When developing an application that sends email, you probably don't want to actually send emails to live email addresses. Laravel provides several ways to "disable" the actual sending of emails during local development.

#### **Log Driver**

Instead of sending your emails, the `log` mail driver will write all email messages to your log files for inspection. Typically, this driver would only be used during local development. For more information on configuring your application per environment, check out the [configuration documentation](https://laravel.com/docs/8.x/configuration#environment-configuration).

#### **HELO / Mailtrap / MailHog**

Finally, you may use a service like [HELO](https://usehelo.com/) or [Mailtrap](https://mailtrap.io/) and the `smtp` driver to send your email messages to a "dummy" mailbox where you may view them in a true email client. This approach has the benefit of allowing you to actually inspect the final emails in Mailtrap's message viewer.

If you are using [Laravel Sail](https://laravel.com/docs/8.x/sail), you may preview your messages using [MailHog](https://github.com/mailhog/MailHog). When Sail is running, you may access the MailHog interface at: `http://localhost:8025`.

## Events

Laravel fires two events during the process of sending mail messages. The `MessageSending` event is fired prior to a message being sent, while the `MessageSent` event is fired after a message has been sent. Remember, these events are fired when the mail is being _sent_, not when it is queued. You may register event listeners for this event in your `App\Providers\EventServiceProvider` service provider:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Mail\Events\MessageSending' => [
        'App\Listeners\LogSendingMessage',
    ],
    'Illuminate\Mail\Events\MessageSent' => [
        'App\Listeners\LogSentMessage',
    ],
];
```

