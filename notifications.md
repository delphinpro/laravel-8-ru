# Уведомления

## Вступление

В дополнение к поддержке [отправки электронной почты](mail.md), Laravel обеспечивает поддержку отправки уведомлений по различным каналам доставки, включая электронную почту, SMS \(через [Vonage](https://www.vonage.com/communications-apis/), ранее известный как Nexmo\), и [Slack](https://slack.com/). Кроме того, [сообществом были созданы различные каналы уведомлений](https://laravel-notification-channels.com/about/#suggesting-a-new-channel) для отправки уведомлений по десяткам различных каналов! Уведомления также могут храниться в базе данных, поэтому они могут отображаться в вашем веб-интерфейсе.

Как правило, уведомления должны быть краткими, информационными сообщениями, которые оповещают пользователей о событиях, произошедших в вашем приложении. Например, если вы пишете приложение для выставления счетов, то можете отправить своим пользователям уведомление "Invoice Paid" по электронной почте и SMS-каналам.

## Генерация уведомлений

В Laravel каждое уведомление представлено отдельным классом, который обычно хранится в каталоге `app/Notifications`. Не волнуйтесь, если вы не увидите этот каталог в вашем приложении — он будет создан, когда вы запустите команду `make:notification`:

```bash
php artisan make:notification InvoicePaid
```

Эта команда поместит новый класс уведомлений в директорию `app/Notifications`. Каждый класс уведомлений содержит метод `via` и различное количество методов построения сообщений, таких как `toMail` или `toDatabase`, которые преобразуют уведомление в сообщение, адаптированное для данного канала.

## Отправка уведомлений

### С использованием трейта Notifiable

Уведомления могут быть отправлены двумя способами: используя метод `notify` из трейта `Notifiable` или с помощью [фасада](facades.md) `Notification`. Трейт `Notifiable` по умолчанию включен в модель `App\Models\User`:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;
}
```

Метод `notify`, предусмотренный этим трейтом, ожидает получения экземпляра уведомления:

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

{% hint style="info" %}
Помните, что в любой из моделей вы можете использовать трейт `Notifiable`. Вы не ограничены включением его только в модель `User`.
{% endhint %}

### С использованием фасада Notification

Кроме того, вы можете отправлять уведомления через [фасад ](facades.md)Notification. Такой подход полезен, когда вам нужно отправить уведомление нескольким уведомляемым субъектам, например, коллекции пользователей. Чтобы отправить уведомления через фасад, передайте все уведомляемые объекты и экземпляр уведомления в метод `send` \(отправить\):

```php
use Illuminate\Support\Facades\Notification;

Notification::send($users, new InvoicePaid($invoice));
```

### Определение каналов доставки

Каждый класс уведомлений имеет метод `via`, который определяет, по каким каналам будет доставляться уведомление. Уведомления могут быть отправлены по каналам `mail`, `database`, `broadcast`, `nexmo` и `slack`.

{% hint style="info" %}
Если Вы хотите использовать другие каналы доставки, например, Telegram или Pusher, посетите веб-сайт сообщества [Laravel Notification Channels](http://laravel-notification-channels.com/).
{% endhint %}

Метод `via` получает экземпляр `$notifiable`, который является экземпляром класса, в который отправляется оповещение. Вы можете использовать `$notifiable`, чтобы определить, по каким каналам должно быть доставлено уведомление:

```php
/**
 * Получить каналы доставки уведомления.
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function via($notifiable)
{
    return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
}
```

### Очередь уведомлений

{% hint style="danger" %}
Перед тем как отправить уведомление в очередь, вы должны настроить вашу очередь и [запустить воркер](queues.md).
{% endhint %}

Отправка уведомлений может занять некоторое время, особенно если каналу необходимо сделать внешний вызов API, чтобы доставить оповещение. Чтобы ускорить время отклика вашего приложения, добавьте к вашему классу интерфейс `ShouldQueue` и трейт `Queueable`, чтобы поставить уведомление в очередь. Интерфейс и трейт уже импортированы для всех уведомлений, генерируемых с помощью команды `make:notification`, поэтому вы можете сразу же добавить их в свой класс уведомлений:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

Если интерфейс `ShouldQueue` был добавлен к вашему уведомлению, вы можете отправить оповещение обычным способом. Laravel обнаружит интерфейс `ShouldQueue` в классе и автоматически поставит в очередь доставку уведомления:

```php
$user->notify(new InvoicePaid($invoice));
```

Если вы хотите задержать доставку уведомления, то можете включить метод `delay`:

```php
$delay = now()->addMinutes(10);

$user->notify((new InvoicePaid($invoice))->delay($delay));
```

Можно передать массив в метод `delay`, чтобы указать величину задержки для конкретных каналов:

```php
$user->notify((new InvoicePaid($invoice))->delay([
    'mail' => now()->addMinutes(5),
    'sms' => now()->addMinutes(10),
]));
```

При отправке уведомлений в очередь для каждого получателя и комбинации каналов будет создано задание в очереди. Например, шесть заданий будут поставлены в очередь, если в вашем уведомлении три получателя и два канала.

#### Настройка соединения очереди уведомлений

По умолчанию, уведомления, поставленные в очередь, будут отправляться по стандартному подключению вашего приложения к очереди. Если вы хотите указать другое соединение, которое должно использоваться для конкретного уведомления, то можете определить свойство `$connection` в классе уведомлений:

```php
/**
 * Имя подключения для использования при постановке уведомления в очередь.
 *
 * @var string
 */
public $connection = 'redis';
```

#### Настройка очередей каналов уведомления

Если вы хотите указать конкретную очередь, которая должна использоваться для каждого канала оповещения, поддерживаемого оповещением, вы можете определить метод `viaQueues` в вашем оповещении. Этот метод должен возвращать массив пар имя канала / имя очереди:

```php
/**
 * Определите, какие очереди следует использовать для каждого канала оповещения.
 *
 * @return array
 */
public function viaQueues()
{
    return [
        'mail' => 'mail-queue',
        'slack' => 'slack-queue',
    ];
}
```

#### **Queued Notifications & Database Transactions**

Когда уведомление в очереди отправляется внутри транзакций БД, оно может быть обработано очередью до того, как транзакция БД будет зафиксирована. В этом случае любые обновления моделей или записей в БД, сделанные вами во время транзакции с БД, могут еще не отразиться в БД. Кроме того, любые модели или записи БД, созданные внутри транзакции, могут не существовать в БД. Если уведомление зависит от этих моделей, то при обработке задания, отправляющего уведомление в очередь, могут возникать неожиданные ошибки.

Если параметр конфигурации соединения в очереди `after_commit` установлен в `false`, вы можете указать, что после фиксации всех открытых транзакций в БД должно быть отправлено определенное уведомление в очереди, определив свойство `$afterCommit` в классе уведомлений:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    public $afterCommit = true;
}
```

{% hint style="info" %}
Чтобы узнать больше о работе с этими проблемами, ознакомьтесь, пожалуйста, с документацией, касающейся [очередей на задания и операций с базами данных](queues.md).
{% endhint %}

### Уведомления по требованию

В некоторых случаях вам может понадобиться отправить уведомление тому, кто не является "пользователем" вашего приложения. Используя метод `route` фасада `Notification`, вы можете указать специальную информацию о маршрутизации уведомлений перед отправкой уведомления:

```php
Notification::route('mail', 'taylor@example.com')
            ->route('nexmo', '5555555555')
            ->route('slack', 'https://hooks.slack.com/services/...')
            ->notify(new InvoicePaid($invoice));
```

## Почтовые уведомления <a id="mail-notifications"></a>

### Форматирование почтовых сообщений

Если уведомление поддерживает отправку по электронной почте, необходимо определить метод `toMail` в классе уведомлений. Этот метод будет получать сущность `$notifiable` и должен возвращать экземпляр `Illuminate\Notifications\Messages\MailMessage`.

Класс `MailMessage` содержит несколько простых методов, которые помогут вам строить транзакционные сообщения электронной почты. Письма могут содержать как строки текста, так и "призыв к действию" \(CTA\). Рассмотрим пример метода `toMail`:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->greeting('Hello!')
                ->line('One of your invoices has been paid!')
                ->action('View Invoice', $url)
                ->line('Thank you for using our application!');
}
```

{% hint style="info" %}
Обратите внимание, что в нашем методе `toMail` мы используем `$this->invoicee->id`. Вы можете передать любые данные, необходимые вашему уведомлению для генерации его сообщения, в конструктор уведомления.
{% endhint %}

В этом примере мы регистрируем приветствие, строку текста, призыв к действию, а затем еще одну строку текста. Эти методы, предоставляемые объектом `MailMessage`, упрощают и ускоряют форматирование небольших транзакционных сообщений. Затем почтовый канал переводит компоненты сообщения в красивый, отзывчивый шаблон HTML-почты с простым текстовым контрагентом. Вот пример письма, сгенерированного `mail` каналом:

![&#x41F;&#x440;&#x438;&#x43C;&#x435;&#x440; &#x441;&#x433;&#x435;&#x43D;&#x435;&#x440;&#x438;&#x440;&#x43E;&#x432;&#x430;&#x43D;&#x43D;&#x43E;&#x433;&#x43E; &#x43F;&#x438;&#x441;&#x44C;&#x43C;&#x430;](.gitbook/assets/notification-example-2.png)

{% hint style="info" %}
При отправке почтовых уведомлений обязательно установите параметр `name` в конфигурационном файле `config/app.php`. Это значение будет использоваться в заголовке и нижнем колонтитуле ваших почтовых уведомлений.
{% endhint %}

#### Другие опции форматирования почтовых уведомлений

Вместо того, чтобы определять "строки" текста в классе уведомлений, можно использовать метод `view` для указания пользовательского шаблона, который должен использоваться для отображения письма-уведомления:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)->view(
        'emails.name', ['invoice' => $this->invoice]
    );
}
```

Вы можете указать обычный текстовый вид для почтового сообщения, передавая имя представления как второй элемент массива, который передается методу `view`:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)->view(
        ['emails.name.html', 'emails.name.plain'],
        ['invoice' => $this->invoice]
    );
}
```

Кроме того, вы можете вернуть полный [почтовый объект](mail.md) из метода `toMail`:

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;

/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return Mailable
 */
public function toMail($notifiable)
{
    return (new InvoicePaidMailable($this->invoice))
                ->to($notifiable->email);
}
```

#### **Сообщения об ошибках**

Некоторые уведомления информируют пользователей об ошибках, например, о неудачной оплате счета-фактуры. Вы можете указать, что почтовое сообщение касается ошибки, вызвав метод `error` при построении сообщения. При использовании метода `error` в почтовом сообщении кнопка действия будет не черной, а красной:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Message
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->error()
                ->subject('Notification Subject')
                ->line('...');
}
```

### Настройка отправителя

По умолчанию, отправитель письма определяется в конфигурационном файле `config/mail.php`. Однако, вы можете указать адрес _from_ для отдельного уведомления, используя метод `from`:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->from('barrett@example.com', 'Barrett Blair')
                ->line('...');
}
```

### Настройка получателя

При отправке уведомлений по `mail` каналу, система уведомлений будет автоматически искать свойство `email` в вашем уведомляемом объекте. Вы можете настроить, какой электронный адрес используется для доставки уведомления, определив метод `routeNotificationForMail` в уведомляемом объекте:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the mail channel.
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return array|string
     */
    public function routeNotificationForMail($notification)
    {
        // Return email address only...
        return $this->email_address;

        // Return email address and name...
        return [$this->email_address => $this->name];
    }
}
```

### Настройка темы сообщения

По умолчанию темой письма является имя класса уведомления, отформатированное в "Title Case". Таким образом, если ваш класс уведомления называется `InvoicePaid`, то темой письма будет `Invoice Paid`. Если вы хотите задать для сообщения другую тему, то можете вызвать метод `subject` при построении сообщения:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->subject('Notification Subject')
                ->line('...');
}
```

### Настройка почтового сервера

По умолчанию, уведомление по электронной почте будет отправлено с помощью почтового сервера, определенного в конфигурационном файле `config/mail.php`. Однако, вы можете указать другой почтовый сервер во время выполнения, вызвав метод `mailer` при создании сообщения:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->mailer('postmark')
                ->line('...');
}
```

### Настройка шаблонов

Вы можете изменить HTML и обычный текстовый шаблон, используемый в почтовых уведомлениях, опубликовав ресурсы пакета уведомлений. После выполнения этой команды шаблоны почтовых уведомлений будут расположены в каталоге `resources/views/vendor/notifications`:

```bash
php artisan vendor:publish --tag=laravel-notifications
```

### Вложения

Чтобы добавлять вложения к почтовому уведомлению, используйте метод `attach` при создании сообщения. В качестве первого аргумента метод `attach` принимает абсолютный путь к файлу:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->greeting('Hello!')
                ->attach('/path/to/file');
}
```

При прикреплении файлов к сообщению, вы также можете указать отображаемое имя и/или MIME тип, передавая массив параметров в качестве второго аргумента в метод `attach`:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->greeting('Hello!')
                ->attach('/path/to/file', [
                    'as' => 'name.pdf',
                    'mime' => 'application/pdf',
                ]);
}
```

В отличие от прикрепления файлов в почтовых объектах, вы не можете прикреплять файл непосредственно с диска хранения с помощью `attachFromStorage`. Лучше использовать метод `attach` с абсолютным путем к файлу на диске хранилища. В качестве альтернативы можно вернуть [почтовый объект](mail.md) из метода `toMail`:

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;

/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return Mailable
 */
public function toMail($notifiable)
{
    return (new InvoicePaidMailable($this->invoice))
                ->to($notifiable->email)
                ->attachFromStorage('/path/to/file');
}
```

#### **Raw Data Attachments**

Метод `attachData` может быть использован для прикрепления необработанной строки байтов в качестве вложения. При вызове метода `attachData` необходимо указать имя файла, который будет присвоен вложению:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->greeting('Hello!')
                ->attachData($this->pdf, 'name.pdf', [
                    'mime' => 'application/pdf',
                ]);
}
```

### Предпросмотр почтовых уведомлений

При проектировании шаблона почтовых уведомлений удобно быстро просмотреть отображенное почтовое сообщение в браузере, как обычный шаблон Blade. По этой причине Laravel позволяет вам возвращать любое почтовое сообщение, сгенерированное почтовым уведомлением, непосредственно из маршрута или контроллера. Когда `MailMessage` возвращается, оно выводится и отображается в браузере, позволяя вам быстро просмотреть его дизайн без необходимости отправлять его на фактический адрес электронной почты:

```php
use App\Models\Invoice;
use App\Notifications\InvoicePaid;

Route::get('/notification', function () {
    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))
                ->toMail($invoice->user);
});
```

## Почтовые уведомления в формате Markdown

Почтовые уведомления в формате Markdown позволяют вам воспользоваться предварительно созданными шаблонами, давая при этом больше свободы в написании более длинных, настраиваемых сообщений. Так как сообщения написаны в Markdown, Laravel может визуализировать красивые, отзывчивые HTML шаблоны для сообщений, одновременно автоматически генерируя контрагент в виде простого текста.

### Генерация сообщения

Для генерации уведомления с соответствующим шаблоном Markdown можно воспользоваться опцией `--markdown` команды `make:notification`:

```bash
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

Как и все остальные почтовые уведомления, уведомления, использующие шаблоны Markdown, должны определять метод `toMail` в своем классе уведомлений. Однако, вместо того, чтобы использовать методы `line` и `action` для построения уведомления, используйте метод `markdown` для указания имени шаблона Markdown, который должен быть использован. В качестве второго аргумента метода может быть передан массив данных, который вы хотите сделать доступным для шаблона:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->subject('Invoice Paid')
                ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

### Написание сообщения

Почтовые уведомления Markdown используют комбинацию компонентов Blade и синтаксис Markdown, что позволяет легко создавать уведомления, используя предварительно разработанные Laravel компоненты уведомлений:

```markup
@component('mail::message')
# Invoice Paid

Your invoice has been paid!

@component('mail::button', ['url' => $url])
View Invoice
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent
```

#### **Button Component**

Компонент кнопки отображает центрированную кнопку с ссылкой. Компонент принимает два аргумента, `url` и необязательный `color`. Поддерживаемые цвета: `primary`, `green` и `red`. Вы можете добавить в уведомление столько компонентов кнопок, сколько пожелаете:

```markup
@component('mail::button', ['url' => $url, 'color' => 'green'])
View Invoice
@endcomponent
```

#### **Panel Component**

Компонент панели отображает заданный блок текста на панели, цвет фона которой немного отличается от цвета фона всего остального уведомления. Это позволяет привлечь внимание к заданному блоку текста:

```markup
@component('mail::panel')
This is the panel content.
@endcomponent
```

#### **Table Component**

Компонент таблицы позволяет преобразовать таблицу Markdown в HTML-таблицу. Компонент принимает Markdown таблицу в качестве ее содержимого. Выравнивание столбцов таблицы поддерживается с использованием стандартного синтаксиса выравнивания Markdown таблиц:

```markup
@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
```

### Настройка компонентов

Вы можете экспортировать все компоненты уведомления Markdown в ваше собственное приложение для настройки. Для экспорта компонентов воспользуйтесь командой `vendor:publish`:

```bash
php artisan vendor:publish --tag=laravel-mail
```

Эта команда опубликует почтовые компоненты Markdown в каталоге `resources/views/vendor/mail`. Директория `mail` будет содержать каталог `html` и `text`, каждый из которых будет содержать свои соответствующие представления каждого доступного компонента. Вы можете настроить эти компоненты как угодно.

#### Настройка CSS

После экспорта компонентов каталог `resources/views/vendor/mail/html/themes` будет содержать файл `default.css`. Вы можете настроить CSS в этом файле, и ваши стили будут автоматически встроены в HTML представление ваших уведомлений Markdown.

Если вы хотите создать совершенно новую тему для компонентов Laravel's Markdown, вы можете поместить CSS-файл в каталог `html/themes`. После именования и сохранения CSS-файла обновите опцию `theme` в файле конфигурации `mail`, чтобы она соответствовала названию вашей новой темы.

Чтобы настроить тему для отдельного уведомления, можно вызвать метод `theme` во время построения почтового сообщения. Метод `theme` принимает имя темы, которое будет использоваться при отправке уведомления:

```php
/**
 * Get the mail representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->theme('invoice')
                ->subject('Invoice Paid')
                ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

## Database Notifications

### Prerequisites

The `database` notification channel stores the notification information in a database table. This table will contain information such as the notification type as well as a JSON data structure that describes the notification.

You can query the table to display the notifications in your application's user interface. But, before you can do that, you will need to create a database table to hold your notifications. You may use the `notifications:table` command to generate a [migration](https://laravel.com/docs/8.x/migrations) with the proper table schema:

```bash
php artisan notifications:table

php artisan migrate
```

### Formatting Database Notifications

If a notification supports being stored in a database table, you should define a `toDatabase` or `toArray` method on the notification class. This method will receive a `$notifiable` entity and should return a plain PHP array. The returned array will be encoded as JSON and stored in the `data` column of your `notifications` table. Let's take a look at an example `toArray` method:

```php
/**
 * Get the array representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function toArray($notifiable)
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

#### **toDatabase Vs. toArray**

The `toArray` method is also used by the `broadcast` channel to determine which data to broadcast to your JavaScript powered frontend. If you would like to have two different array representations for the `database` and `broadcast` channels, you should define a `toDatabase` method instead of a `toArray` method.

### Accessing The Notifications

Once notifications are stored in the database, you need a convenient way to access them from your notifiable entities. The `Illuminate\Notifications\Notifiable` trait, which is included on Laravel's default `App\Models\User` model, includes a `notifications` [Eloquent relationship](https://laravel.com/docs/8.x/eloquent-relationships) that returns the notifications for the entity. To fetch notifications, you may access this method like any other Eloquent relationship. By default, notifications will be sorted by the `created_at` timestamp with the most recent notifications at the beginning of the collection:

```php
$user = App\Models\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

If you want to retrieve only the "unread" notifications, you may use the `unreadNotifications` relationship. Again, these notifications will be sorted by the `created_at` timestamp with the most recent notifications at the beginning of the collection:

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> To access your notifications from your JavaScript client, you should define a notification controller for your application which returns the notifications for a notifiable entity, such as the current user. You may then make an HTTP request to that controller's URL from your JavaScript client.

### Marking Notifications As Read

Typically, you will want to mark a notification as "read" when a user views it. The `Illuminate\Notifications\Notifiable` trait provides a `markAsRead` method, which updates the `read_at` column on the notification's database record:

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```

However, instead of looping through each notification, you may use the `markAsRead` method directly on a collection of notifications:

```php
$user->unreadNotifications->markAsRead();
```

You may also use a mass-update query to mark all of the notifications as read without retrieving them from the database:

```php
$user = App\Models\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```

You may `delete` the notifications to remove them from the table entirely:

```php
$user->notifications()->delete();
```

## Broadcast Notifications

### Prerequisites

Before broadcasting notifications, you should configure and be familiar with Laravel's [event broadcasting](https://laravel.com/docs/8.x/broadcasting) services. Event broadcasting provides a way to react to server-side Laravel events from your JavaScript powered frontend.

### Formatting Broadcast Notifications

The `broadcast` channel broadcasts notifications using Laravel's [event broadcasting](https://laravel.com/docs/8.x/broadcasting) services, allowing your JavaScript powered frontend to catch notifications in realtime. If a notification supports broadcasting, you can define a `toBroadcast` method on the notification class. This method will receive a `$notifiable` entity and should return a `BroadcastMessage` instance. If the `toBroadcast` method does not exist, the `toArray` method will be used to gather the data that should be broadcast. The returned data will be encoded as JSON and broadcast to your JavaScript powered frontend. Let's take a look at an example `toBroadcast` method:

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * Get the broadcastable representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return BroadcastMessage
 */
public function toBroadcast($notifiable)
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

#### **Broadcast Queue Configuration**

All broadcast notifications are queued for broadcasting. If you would like to configure the queue connection or queue name that is used to queue the broadcast operation, you may use the `onConnection` and `onQueue` methods of the `BroadcastMessage`:

```php
return (new BroadcastMessage($data))
                ->onConnection('sqs')
                ->onQueue('broadcasts');
```

#### **Customizing The Notification Type**

In addition to the data you specify, all broadcast notifications also have a `type` field containing the full class name of the notification. If you would like to customize the notification `type`, you may define a `broadcastType` method on the notification class:

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * Get the type of the notification being broadcast.
 *
 * @return string
 */
public function broadcastType()
{
    return 'broadcast.message';
}
```

### Listening For Notifications

Notifications will broadcast on a private channel formatted using a `{notifiable}.{id}` convention. So, if you are sending a notification to an `App\Models\User` instance with an ID of `1`, the notification will be broadcast on the `App.Models.User.1` private channel. When using [Laravel Echo](https://laravel.com/docs/8.x/broadcasting), you may easily listen for notifications on a channel using the `notification` method:

```javascript
Echo.private('App.Models.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```

#### **Customizing The Notification Channel**

If you would like to customize which channel that an entity's broadcast notifications are broadcast on, you may define a `receivesBroadcastNotificationsOn` method on the notifiable entity:

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The channels the user receives notification broadcasts on.
     *
     * @return string
     */
    public function receivesBroadcastNotificationsOn()
    {
        return 'users.'.$this->id;
    }
}
```

## SMS Notifications

### Prerequisites

Sending SMS notifications in Laravel is powered by [Vonage](https://www.vonage.com/) \(formerly known as Nexmo\). Before you can send notifications via Vonage, you need to install the `laravel/nexmo-notification-channel` and `nexmo/laravel` Composer packages

```bash
composer require laravel/nexmo-notification-channel nexmo/laravel
```

The `nexmo/laravel` package includes [its own configuration file](https://github.com/Nexmo/nexmo-laravel/blob/master/config/nexmo.php). However, you are not required to export this configuration file to your own application. You can simply use the `NEXMO_KEY` and `NEXMO_SECRET` environment variables to set your Vonage public and secret key.

Next, you will need to add a `nexmo` configuration entry to your `config/services.php` configuration file. You may copy the example configuration below to get started:

```php
'nexmo' => [
    'sms_from' => '15556666666',
],
```

The `sms_from` option is the phone number that your SMS messages will be sent from. You should generate a phone number for your application in the Vonage control panel.

### Formatting SMS Notifications

If a notification supports being sent as an SMS, you should define a `toNexmo` method on the notification class. This method will receive a `$notifiable` entity and should return an `Illuminate\Notifications\Messages\NexmoMessage` instance:

```php
/**
 * Get the Vonage / SMS representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\NexmoMessage
 */
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your SMS message content');
}
```

#### **Unicode Content**

If your SMS message will contain unicode characters, you should call the `unicode` method when constructing the `NexmoMessage` instance:

```php
/**
 * Get the Vonage / SMS representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\NexmoMessage
 */
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your unicode message')
                ->unicode();
}
```

### Formatting Shortcode Notifications

Laravel also supports sending shortcode notifications, which are pre-defined message templates in your Vonage account. To send a shortcode SMS notification, you should define a `toShortcode` method on your notification class. From within this method, you may return an array specifying the type of notification \(`alert`, `2fa`, or `marketing`\) as well as the custom values that will populate the template:

```php
/**
 * Get the Vonage / Shortcode representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function toShortcode($notifiable)
{
    return [
        'type' => 'alert',
        'custom' => [
            'code' => 'ABC123',
        ];
    ];
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> Like [routing SMS Notifications](https://laravel.com/docs/8.x/notifications#routing-sms-notifications), you should implement the `routeNotificationForShortcode` method on your notifiable model.

### Customizing The "From" Number

If you would like to send some notifications from a phone number that is different from the phone number specified in your `config/services.php` file, you may call the `from` method on a `NexmoMessage` instance:

```php
/**
 * Get the Vonage / SMS representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return NexmoMessage
 */
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your SMS message content')
                ->from('15554443333');
}
```

### Routing SMS Notifications

To route Vonage notifications to the proper phone number, define a `routeNotificationForNexmo` method on your notifiable entity:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Nexmo channel.
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return string
     */
    public function routeNotificationForNexmo($notification)
    {
        return $this->phone_number;
    }
}
```

## Slack Notifications

### Prerequisites

Before you can send notifications via Slack, you must install the Slack notification channel via Composer:

```bash
composer require laravel/slack-notification-channel
```

You will also need to configure an ["Incoming Webhook"](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) integration for your Slack team. This integration will provide you with a URL you may use when [routing Slack notifications](https://laravel.com/docs/8.x/notifications#routing-slack-notifications).

### Formatting Slack Notifications

If a notification supports being sent as a Slack message, you should define a `toSlack` method on the notification class. This method will receive a `$notifiable` entity and should return an `Illuminate\Notifications\Messages\SlackMessage` instance. Slack messages may contain text content as well as an "attachment" that formats additional text or an array of fields. Let's take a look at a basic `toSlack` example:

```php
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Message\SlackMessage
 */
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->content('One of your invoices has been paid!');
}
```

#### **Customizing The Sender & Recipient**

You may use the `from` and `to` methods to customize the sender and recipient. The `from` method accepts a username and emoji identifier, while the `to` method accepts a channel or username:

```php
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\SlackMessage
 */
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->from('Ghost', ':ghost:')
                ->to('#bots')
                ->content('This will be sent to #bots');
}
```

You may also use an image as your from "logo" instead of an emoji:

```php
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\SlackMessage
 */
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->from('Laravel')
                ->image('https://laravel.com/img/favicon/favicon.ico')
                ->content('This will display the Laravel logo next to the message');
}
```

### Slack Attachments

You may also add "attachments" to Slack messages. Attachments provide richer formatting options than simple text messages. In this example, we will send an error notification about an exception that occurred in an application, including a link to view more details about the exception:

```php
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/exceptions/'.$this->exception->id);

    return (new SlackMessage)
                ->error()
                ->content('Whoops! Something went wrong.')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Exception: File Not Found', $url)
                               ->content('File [background.jpg] was not found.');
                });
}
```

Attachments also allow you to specify an array of data that should be presented to the user. The given data will be presented in a table-style format for easy reading:

```php
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/invoices/'.$this->invoice->id);

    return (new SlackMessage)
                ->success()
                ->content('One of your invoices has been paid!')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Invoice 1322', $url)
                               ->fields([
                                    'Title' => 'Server Expenses',
                                    'Amount' => '$1,234',
                                    'Via' => 'American Express',
                                    'Was Overdue' => ':-1:',
                                ]);
                });
}
```

#### **Markdown Attachment Content**

If some of your attachment fields contain Markdown, you may use the `markdown` method to instruct Slack to parse and display the given attachment fields as Markdown formatted text. The values accepted by this method are: `pretext`, `text`, and / or `fields`. For more information about Slack attachment formatting, check out the [Slack API documentation](https://api.slack.com/docs/message-formatting#message_formatting):

```php
/**
 * Get the Slack representation of the notification.
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/exceptions/'.$this->exception->id);

    return (new SlackMessage)
                ->error()
                ->content('Whoops! Something went wrong.')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('Exception: File Not Found', $url)
                               ->content('File [background.jpg] was *not found*.')
                               ->markdown(['text']);
                });
}
```

### Routing Slack Notifications

To route Slack notifications to the proper Slack team and channel, define a `routeNotificationForSlack` method on your notifiable entity. This should return the webhook URL to which the notification should be delivered. Webhook URLs may be generated by adding an "Incoming Webhook" service to your Slack team:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return string
     */
    public function routeNotificationForSlack($notification)
    {
        return 'https://hooks.slack.com/services/...';
    }
}
```

## Localizing Notifications

Laravel allows you to send notifications in a locale other than the HTTP request's current locale, and will even remember this locale if the notification is queued.

To accomplish this, the `Illuminate\Notifications\Notification` class offers a `locale` method to set the desired language. The application will change into this locale when the notification is being evaluated and then revert back to the previous locale when evaluation is complete:

```php
$user->notify((new InvoicePaid($invoice))->locale('es'));
```

Localization of multiple notifiable entries may also be achieved via the `Notification` facade:

```php
Notification::locale('es')->send(
    $users, new InvoicePaid($invoice)
);
```

### User Preferred Locales

Sometimes, applications store each user's preferred locale. By implementing the `HasLocalePreference` contract on your notifiable model, you may instruct Laravel to use this stored locale when sending a notification:

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

Once you have implemented the interface, Laravel will automatically use the preferred locale when sending notifications and mailables to the model. Therefore, there is no need to call the `locale` method when using this interface:

```php
$user->notify(new InvoicePaid($invoice));
```

## Notification Events

When a notification is sent, the `Illuminate\Notifications\Events\NotificationSent` [event](https://laravel.com/docs/8.x/events) is fired by the notification system. This contains the "notifiable" entity and the notification instance itself. You may register listeners for this event in your `EventServiceProvider`:

```php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Notifications\Events\NotificationSent' => [
        'App\Listeners\LogNotification',
    ],
];
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> After registering listeners in your `EventServiceProvider`, use the `event:generate` Artisan command to quickly generate listener classes.

Within an event listener, you may access the `notifiable`, `notification`, and `channel` properties on the event to learn more about the notification recipient or the notification itself:

```php
/**
 * Handle the event.
 *
 * @param  \Illuminate\Notifications\Events\NotificationSent  $event
 * @return void
 */
public function handle(NotificationSent $event)
{
    // $event->channel
    // $event->notifiable
    // $event->notification
    // $event->response
}
```

## Custom Channels

Laravel ships with a handful of notification channels, but you may want to write your own drivers to deliver notifications via other channels. Laravel makes it simple. To get started, define a class that contains a `send` method. The method should receive two arguments: a `$notifiable` and a `$notification`.

Within the `send` method, you may call methods on the notification to retrieve a message object understood by your channel and then send the notification to the `$notifiable` instance however you wish:

```php
<?php

namespace App\Channels;

use Illuminate\Notifications\Notification;

class VoiceChannel
{
    /**
     * Send the given notification.
     *
     * @param  mixed  $notifiable
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return void
     */
    public function send($notifiable, Notification $notification)
    {
        $message = $notification->toVoice($notifiable);

        // Send notification to the $notifiable instance...
    }
}
```

Once your notification channel class has been defined, you may return the class name from the `via` method of any of your notifications. In this example, the `toVoice` method of your notification can return whatever object you choose to represent voice messages. For example, you might define your own `VoiceMessage` class to represent these messages:

```php
<?php

namespace App\Notifications;

use App\Channels\Messages\VoiceMessage;
use App\Channels\VoiceChannel;
use Illuminate\Bups\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification
{
    use Queueable;

    /**
     * Get the notification channels.
     *
     * @param  mixed  $notifiable
     * @return array|string
     */
    public function via($notifiable)
    {
        return [VoiceChannel::class];
    }

    /**
     * Get the voice representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return VoiceMessage
     */
    public function toVoice($notifiable)
    {
        // ...
    }
}
```

