# Структура директорий

## Вступление

Структура приложения Laravel по умолчанию предназначена для обеспечения отличной отправной точки как для больших, так и для маленьких приложений. Но вы можете организовать свое приложение так, как вам нравится. Laravel практически не накладывает никаких ограничений на то, где находится тот или иной класс — до тех пор, пока Composer может автозагрузить класс.

## Корневая директория

### Директория App

Каталог `app` содержит основной код вашего приложения. В ближайшее время мы рассмотрим этот каталог более подробно, однако почти все классы вашего приложения будут находиться в этом каталоге.

### Директория Bootstrap

Каталог `bootstrap` содержит файл `app.php`, который загружает фреймворк. Этот каталог также содержит каталог `cache`, который содержит сгенерированные фреймворком файлы для оптимизации производительности, такие как маршрутные и сервисные кэш-файлы. Обычно нет необходимости изменять какие-либо файлы в этом каталоге.

### Директория Config

Каталог `config`, как следует из названия, содержит все конфигурационные файлы вашего приложения. Здесь вы можете прочитать все эти файлы и ознакомиться со всеми доступными вам опциями.

### Директория Database

В каталоге `database` содержатся миграции вашей базы данных, фабрики моделей и сидеры. При желании вы также можете использовать этот каталог для хранения БД SQLite.

### Директория Public

Каталог `public` содержит файл `index.php`, который является точкой входа для всех запросов, поступающих в ваше приложение, и настраивает автозагрузку. Этот каталог также содержит ваши ассеты, такие как изображения, JavaScript и CSS.

### Директория Resources

Каталог `resources` содержит ваши представления, а также ваши необработанные, некомпилированные активы, такие как CSS или JavaScript. В этом каталоге также находятся все ваши языковые файлы.

### Директория Routes

Каталог `routes` \(маршруты\) содержит все определения маршрутов для вашего приложения. По умолчанию в Laravel включено несколько файлов с маршрутами: `web.php`, `api.php`, `console.php` и `channels.php`.

Файл `web.php` содержит маршруты, которые `RouteServiceProvider` помещает в группу посредников `web`, обеспечивающую состояние сессии, защиту от CSRF и шифрование cookie-файлов. Если ваше приложение не предоставляет RESTful API, то вполне вероятно, что все ваши маршруты будут определены в файле `web.php`.

Файл `api.php` содержит маршруты, которые `RouteServiceProvider` помещает в группу посредников `api`. Эти маршруты предназначены для запросов без состояния, поэтому запросы, поступающие в приложение по этим маршрутам, предназначены для аутентификации с помощью токенов и не будут иметь доступа к сессии.

В файле `console.php` вы можете определить все ваши консольные команды, основанные на Closure. Каждый Closure привязан к экземпляру команды, что обеспечивает простой подход к взаимодействию с IO-методами каждой команды. Хотя этот файл не определяет HTTP-маршруты, он задаёт консольные точки входа \(маршруты\) в ваше приложение.

В файле `channels.php` вы можете зарегистрировать все каналы [трансляции событий](../translyaciya.md), которые поддерживает ваше приложение.

### Директория Storage

Каталог `storage` содержит ваши журналы, скомпилированные шаблоны Blade, файловые сессии, файловый кэш и другие файлы, сгенерированные фреймворком. Этот каталог разделен на каталоги `app`, `framework` и `logs`. Каталог `app` может использоваться для хранения любых файлов, сгенерированных вашим приложением. Каталог `framework` используется для хранения файлов и кэшей, сгенерированных фреймворком. Наконец, каталог `logs` содержит файлы журналов вашего приложения.

Каталог `storage/app/public` может использоваться для хранения пользовательских файлов, таких как профильные аватары, которые должны быть общедоступны. Вы должны создать в `public/storage` символическую ссылку, указывающую на этот каталог. Вы можете создать ссылку, используя команду `php artisan storage:link`.

### Директория Tests

Каталог `tests` содержит ваши автоматизированные тесты. Примеры модульных и функциональных тестов PHPUnit предоставляются из коробки. Каждый класс теста должен быть дополнен словом `Test`. Вы можете запускать свои тесты, используя команды `phpunit` или `php vendor/bin/phpunit`. Или, если вы хотите получить более подробное и красивое представление о результатах вашего теста, вы можете запустить свои тесты с помощью команды Artisan `php artisan test`.

### Директория Vendor

Каталог `vendor` содержит ваши [Composer](https://getcomposer.org/)-зависимости.

## The App Directory

Большая часть вашего приложения находится в каталоге `app`. По умолчанию этот каталог находится в пространстве имен `App` и автоматически загружается Composer с помощью [стандарта автозагрузки PSR-4](https://www.php-fig.org/psr/psr-4/).

Каталог `app` содержит множество дополнительных каталогов, таких как `Console`, `Http` и `Providers`. Считайте, что каталоги `Console` и `Http` предоставляют API в ядре вашего приложения. Протоколы HTTP и CLI являются механизмами взаимодействия с вашим приложением, но на самом деле не содержат логики приложения. Другими словами, они являются двумя способами выдачи команд вашему приложению. Каталог `Console` содержит все ваши команды Artisan, в то время как каталог `Http` содержит ваши контроллеры, промежуточное ПО и запросы.

Внутри каталога `app` будет сгенерирован ряд других каталогов, так как вы используете команды `make` для генерации классов. Так, например, каталог `app/Jobs` не будет существовать до тех пор, пока вы не выполните команду `make:job` для генерации job-класса.

{% hint style="info" %}
Многие из классов в каталоге `app` могут быть сгенерированы Artisan с помощью команд. Чтобы просмотреть доступные команды, запустите команду `php artisan list make` в своем терминале.
{% endhint %}

### The Broadcasting Directory

Каталог `broadcasting` содержит все классы широковещательных каналов для вашего приложения. Эти классы генерируются с помощью команды `make:channel`. Этот каталог не существует по умолчанию, но будет создан при создании первого канала. Чтобы узнать больше о каналах, ознакомьтесь с документацией по [трансляции событий](../translyaciya.md).

### The Console Directory

Каталог `Console` содержит все пользовательские команды Artisan для вашего приложения. Эти команды могут быть сгенерированы с помощью команды `make:command`. В этом каталоге также находится ядро консоли, в котором зарегистрированы пользовательские команды Artisan и определены [запланированные задачи](../scheduling.md).

### The Events Directory

Этот каталог не существует по умолчанию, но будет создан для вас командами `event:generate` и `make:event`. В каталоге `Events` находятся [классы событий](../events.md). События могут использоваться для оповещения других частей вашего приложения о том, что данное действие произошло, обеспечивая большую гибкость и развязку.

### The Exceptions Directory

Каталог `Exceptions` содержит обработчик исключений вашего приложения, а также является хорошим местом для размещения любых исключений, выбрасываемых вашим приложением. Если вы хотите настроить то, как ваши исключения записываются в журнал или выводятся, вы должны изменить класс `Handler` в этом каталоге.

### The Http Directory

Каталог `Http` содержит ваши контроллеры, посредники и запросы форм. Почти вся логика обработки запросов, поступающих в Ваше приложение, будет размещена в этом каталоге.

### The Jobs Directory

Этот каталог не существует по умолчанию, но будет создан, если вы выполните команду `make:job`. В каталоге `Jobs` хранятся задания, предназначенные для работы в [очереди](../queues.md). Задания могут быть поставлены в очередь вашим приложением или выполняться синхронно в пределах текущего жизненного цикла запроса. Задания, которые выполняются синхронно во время текущего запроса, иногда называют "командами", так как они являются реализацией [паттерна команды](https://ru.wikipedia.org/wiki/Команда_%28шаблон_проектирования%29).

### The Listeners Directory

Этот каталог не существует по умолчанию, но будет создан, если вы выполните команды `event:generate` или `make:listener`. Каталог `Listeners` содержит классы, которые обрабатывают ваши [события](../events.md). Слушатели событий получают экземпляр события и выполняют логику в ответ на запускаемое событие. Например, событие `UserRegistered` может быть обработано слушателем `SendWelcomeEmail`.

### The Mail Directory

Этот каталог не существует по умолчанию, но будет создан, если вы выполните команду `make:mail`. Директория `Mail` содержит все ваши [классы, которые представляют письма](../mail.md), отправляемые вашим приложением. Объекты Mail позволяют вам инкапсулировать всю логику построения письма в один, простой класс, который может быть отправлен с помощью метода `Mail::send`.

### The Models Directory

Каталог `Models` содержит все [классы моделей Eloquent](../eloquent-1.md). Eloquent ORM, входящий в состав Laravel, обеспечивает простую и удобную реализацию ActiveRecord для работы с вашей базой данных. Каждая таблица базы данных имеет соответствующую "Модель", которая используется для взаимодействия с этой таблицей. Модели позволяют запрашивать данные в ваших таблицах, а также вставлять новые записи в таблицу.

### The Notifications Directory

Этот каталог не существует по умолчанию, но будет создан, если вы выполните команду `make:notification`. Директория `Notifications` содержит все "транзакционные" [уведомления](../notifications.md), которые отправляются Вашим приложением, например, простые уведомления о событиях, происходящих внутри Вашего приложения. Функции уведомления Laravel абстрагируются от отправки уведомлений различным драйверам, таким как электронная почта, Slack, SMS, или хранятся в базе данных.

### The Policies Directory

Этот каталог не существует по умолчанию, но будет создан, если вы выполните команду `make:policy`. Каталог `Policies` содержит [классы политики авторизации](../security/authorization.md) для вашего приложения. Политики используются для определения того, может ли пользователь выполнять определенное действие в отношении ресурса.

### The Providers Directory

The `Providers` directory contains all of the [service providers](../providers.md) for your application. Service providers bootstrap your application by binding services in the service container, registering events, or performing any other tasks to prepare your application for incoming requests.

In a fresh Laravel application, this directory will already contain several providers. You are free to add your own providers to this directory as needed.

### The Rules Directory

Этот каталог не существует по умолчанию, но будет создан, если вы выполните команду `make:rule`. Директория `Rules` содержит объекты пользовательского правила валидации для вашего приложения. Правила используются для инкапсуляции сложной логики валидации в простой объект. Для получения более подробной информации обратитесь к [документации по валидации](../validation.md).

