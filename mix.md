# Компиляция ассетов \(Mix\)

## Вступление

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix), пакет, разработанный создателем [Laracasts](https://laracasts.com/) Джеффри Уэем \(Jeffrey Way\), предоставляет API для определения этапов компиляции [webpack](https://webpack.js.org/) для вашего приложения Laravel с использованием нескольких распространенных препроцессоров CSS и JavaScript.

Другими словами, Mix делает компиляцию и минимизацию CSS- и JavaScript-файлов вашего приложения простой задачей. С помощью последовательности простых методов вы можете легко и быстро определить конвейер ваших ресурсов. Например:

```javascript
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

Если вы когда-нибудь были запутаны и перегружены началом работы с webpack и компиляцией ресурсов, вы будете в восторге от Laravel Mix. Тем не менее, от вас не требуется использовать его при разработке приложения; вы можете свободно использовать любой инструмент для работы с ресурсами по вашему желанию, или даже не использовать его вообще.

{% hint style="info" %}
Если вам нужно построить свое приложение с помощью Laravel и [Tailwind CSS](https://tailwindcss.com/), ознакомьтесь с одним из наших [стартовых пакетов для приложений](getting-started/starter-kits.md).
{% endhint %}

## Установка и настройка

#### **Установка Node**JS

Перед запуском Mix необходимо сначала убедиться, что на машине установлены Node.js и NPM:

```bash
node -v
npm -v
```

Вы можете легко установить последнюю версию Node и NPM с помощью графических инсталляторов с [официального сайта Node](https://nodejs.org/en/download/). Или, если вы используете [Laravel Sail](packages/sail.md), вы можете вызвать Node и NPM через Sail:

```bash
./sail node -v
./sail npm -v
```

#### **Установка Laravel Mix**

Остается только установить Laravel Mix. В новой установке Laravel вы найдете файл `package.json` в каталоге, расположенном в корне структуры вашего приложения. Файл `package.json` по умолчанию уже содержит все, что вам нужно для начала работы с Laravel Mix. Думайте об этом файле, как о вашем файле `composer.json`, за исключением того, что он определяет зависимости Node вместо зависимостей PHP. Вы можете установить зависимости, на которые он ссылается, выполнив команду:

```bash
npm install
```

## Запуск Mix

Mix является конфигурационным слоем поверх [webpack](https://webpack.js.org/), поэтому для выполнения задач Mix вам нужно выполнить только один из сценариев NPM, который включен в стандартный файл Laravel `package.json`. Когда вы запускаете сценарии разработки или производства, все CSS и JavaScript ресурсы вашего приложения будут скомпилированы и помещены в каталог `public` вашего приложения:

```bash
// Выполнить все задачи Mix...
npm run dev

// Выполнить все задачи Mix и минифицировать ресурсы...
npm run prod
```

#### **Наблюдение за изменениями ресурсов**

Команда `npm run watch` будет продолжать работать в вашем терминале и отслеживать все соответствующие CSS и JavaScript-файлы на наличие изменений. Webpack автоматически перекомпилирует Ваши ресурсы при обнаружении изменений в одном из этих файлов:

```bash
npm run watch
```

Webpack может быть не в состоянии обнаружить изменения ваших файлов в определенных локальных средах разработки. Если это происходит в Вашей системе, подумайте об использовании команды `watch-poll`:

```bash
npm run watch-poll
```

## Работа с таблицами стилей

Файл `webpack.mix.js` вашего приложения является точкой входа для всей компиляции ресурсов. Думайте об этом, как о легкой обертке вокруг конфигурации [webpack](https://webpack.js.org/). Задачи Mix могут быть сцеплены вместе, чтобы точно определить, как ваши ресурсы должны быть скомпилированы.

### Tailwind CSS

[Tailwind CSS](https://tailwindcss.com/) — это современный фреймворк для создания сайтов. Давайте покопаемся в том, как начать использовать его в проекте с Laravel Mix. Сначала мы должны установить Tailwind с помощью NPM и сгенерировать наш конфигурационный файл Tailwind:

```bash
npm install

npm install -D tailwindcss

npx tailwindcss init
```

Команда `init` сгенерирует файл `tailwind.config.js`. В этом файле вы можете настроить пути к всем шаблонам вашего приложения и JavaScript, чтобы Tailwind мог исключать неиспользуемые стили при оптимизации CSS для производства:

```javascript
purge: [
    './storage/framework/views/*.php',
    './resources/**/*.blade.php',
    './resources/**/*.js',
    './resources/**/*.vue',
],
```

Далее необходимо добавить каждый из "слоев" Tailwind в файл `resources/css/app.css` вашего приложения:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

После того, как вы настроили слои Tailwind, вы готовы обновить файл `webpack.mix.js` вашего приложения для компиляции CSS, работающего на Tailwind:

```javascript
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css', [
        require('tailwindcss'),
    ]);
```

Наконец, вы должны указать таблицу стилей в шаблоне основной раскладки вашего приложения. Многие приложения предпочитают хранить этот шаблон в `resources/views/layouts/app.blade.php`. Кроме того, убедитесь, что вы добавили метатег viewport `meta`, если его еще нет:

```markup
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link href="/css/app.css" rel="stylesheet">
</head>
```

### PostCSS

[PostCSS](https://postcss.org/), мощный инструмент для преобразования вашего CSS, входит в комплект поставки Laravel Mix. По умолчанию Mix использует популярный плагин [Autoprefixer](https://github.com/postcss/autoprefixer) для автоматического применения всех необходимых вендорных префиксов CSS3. Однако, вы можете добавлять любые дополнительные плагины, необходимые для вашего приложения.

Сначала установите нужный плагин через NPM и включите его в массив плагинов при вызове метода `postCss` Mix. Метод `postCss` принимает в качестве первого аргумента путь к вашему CSS-файлу, а в качестве второго аргумента — директорию, в которую должен быть помещён скомпилированный файл:

```javascript
mix.postCss('resources/css/app.css', 'public/css', [
    require('postcss-custom-properties')
]);
```

Или вы можете выполнить `postCss` без дополнительных плагинов для достижения простой CSS компиляции и минификации:

```javascript
mix.postCss('resources/css/app.css', 'public/css');
```

### Sass

Метод `sass` позволяет скомпилировать [Sass](https://sass-lang.com/) в CSS, который понятен веб-браузерам. Метод `sass` принимает в качестве первого аргумента путь к вашему файлу Sass, а в качестве второго аргумента — директорию, в которую должен быть помещен скомпилированный файл:

```javascript
mix.sass('resources/sass/app.scss', 'public/css');
```

Вы можете скомпилировать несколько Sass-файлов в соответствующие CSS-файлы и даже настроить выходную директорию результирующего CSS, многократно вызывая метод `sass`:

```javascript
mix.sass('resources/sass/app.sass', 'public/css')
    .sass('resources/sass/admin.sass', 'public/css/admin');
```

### Обработка URL

Поскольку Laravel Mix построен на базе webpack'а, важно понимать несколько концепций webpack'а. Для CSS компиляции, webpack перепишет и оптимизирует любые вызовы `url()` в ваших таблицах стилей. Хотя первоначально это может звучать странно, но это невероятно мощная часть функциональности. Представьте, что мы хотим скомпилировать Sass, который включает в себя относительный URL к изображению:

```css
.example {
    background: url('../images/example.png');
}
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> Absolute paths for any given `url()` will be excluded from URL-rewriting. For example, `url('/images/thing.png')` or `url('http://example.com/images/thing.png')` won't be modified.

By default, Laravel Mix and webpack will find `example.png`, copy it to your `public/images` folder, and then rewrite the `url()` within your generated stylesheet. As such, your compiled CSS will be:

```css
.example {
    background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
}
```

As useful as this feature may be, it's possible that your existing folder structure is already configured in a way you like. If this is the case, you may disable `url()` rewriting like so:

```javascript
mix.sass('resources/sass/app.scss', 'public/css').options({
    processCssUrls: false
});
```

With this addition to your `webpack.mix.js` file, Mix will no longer match any `url()` or copy assets to your public directory. In other words, the compiled CSS will look just like how you originally typed it:

```css
.example {
    background: url("../images/thing.png");
}
```

### Source Maps

Though disabled by default, source maps may be activated by calling the `mix.sourceMaps()` method in your `webpack.mix.js` file. Though it comes with a compile/performance cost, this will provide extra debugging information to your browser's developer tools when using compiled assets:

```javascript
mix.js('resources/js/app.js', 'public/js')
    .sourceMaps();
```

**Style Of Source Mapping**

Webpack offers a variety of [source mapping styles](https://webpack.js.org/configuration/devtool/#devtool). By default, Mix's source mapping style is set to `eval-source-map`, which provides a fast rebuild time. If you want to change the mapping style, you may do so using the `sourceMaps` method:

```javascript
let productionSourceMaps = false;

mix.js('resources/js/app.js', 'public/js')
    .sourceMaps(productionSourceMaps, 'source-map');
```

## Working With JavaScript

Mix provides several features to help you work with your JavaScript files, such as compiling modern ECMAScript, module bundling, minification, and concatenating plain JavaScript files. Even better, this all works seamlessly, without requiring an ounce of custom configuration:

```javascript
mix.js('resources/js/app.js', 'public/js');
```

With this single line of code, you may now take advantage of:

* The latest EcmaScript syntax.
* Modules
* Minification for production environments.

### Vue

Mix will automatically install the Babel plugins necessary for Vue single-file component compilation support when using the `vue` method. No further configuration is required:

```javascript
mix.js('resources/js/app.js', 'public/js')
   .vue();
```

Once your JavaScript has been compiled, you can reference it in your application:

```markup
<head>
    <!-- ... -->

    <script src="/js/app.js"></script>
</head>
```

### React

Mix can automatically install the Babel plugins necessary for React support. To get started, add a call to the `react` method:

```javascript
mix.js('resources/js/app.jsx', 'public/js')
   .react();
```

Behind the scenes, Mix will download and include the appropriate `babel-preset-react` Babel plugin. Once your JavaScript has been compiled, you can reference it in your application:

```markup
<head>
    <!-- ... -->

    <script src="/js/app.js"></script>
</head>
```

### Vendor Extraction

One potential downside to bundling all of your application-specific JavaScript with your vendor libraries such as React and Vue is that it makes long-term caching more difficult. For example, a single update to your application code will force the browser to re-download all of your vendor libraries even if they haven't changed.

If you intend to make frequent updates to your application's JavaScript, you should consider extracting all of your vendor libraries into their own file. This way, a change to your application code will not affect the caching of your large `vendor.js` file. Mix's `extract` method makes this a breeze:

```javascript
mix.js('resources/js/app.js', 'public/js')
    .extract(['vue'])
```

The `extract` method accepts an array of all libraries or modules that you wish to extract into a `vendor.js` file. Using the snippet above as an example, Mix will generate the following files:

* `public/js/manifest.js`: _The Webpack manifest runtime_
* `public/js/vendor.js`: _Your vendor libraries_
* `public/js/app.js`: _Your application code_

To avoid JavaScript errors, be sure to load these files in the proper order:

```markup
<script src="/js/manifest.js"></script>
<script src="/js/vendor.js"></script>
<script src="/js/app.js"></script>
```

### Custom Webpack Configuration

Occasionally, you may need to manually modify the underlying Webpack configuration. For example, you might have a special loader or plugin that needs to be referenced.

Mix provides a useful `webpackConfig` method that allows you to merge any short Webpack configuration overrides. This is particularly appealing, as it doesn't require you to copy and maintain your own copy of the `webpack.config.js` file. The `webpackConfig` method accepts an object, which should contain any [Webpack-specific configuration](https://webpack.js.org/configuration/) that you wish to apply.

```javascript
mix.webpackConfig({
    resolve: {
        modules: [
            path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
        ]
    }
});
```

## Versioning / Cache Busting

Many developers suffix their compiled assets with a timestamp or unique token to force browsers to load the fresh assets instead of serving stale copies of the code. Mix can automatically handle this for you using the `version` method.

The `version` method will append a unique hash to the filenames of all compiled files, allowing for more convenient cache busting:

```javascript
mix.js('resources/js/app.js', 'public/js')
    .version();
```

After generating the versioned file, you won't know the exact filename. So, you should use Laravel's global `mix` function within your [views](https://laravel.com/docs/8.x/views) to load the appropriately hashed asset. The `mix` function will automatically determine the current name of the hashed file:

```markup
<script src="{{ mix('/js/app.js') }}"></script>
```

Because versioned files are usually unnecessary in development, you may instruct the versioning process to only run during `npm run prod`:

```javascript
mix.js('resources/js/app.js', 'public/js');

if (mix.inProduction()) {
    mix.version();
}
```

**Custom Mix Base URLs**

If your Mix compiled assets are deployed to a CDN separate from your application, you will need to change the base URL generated by the `mix` function. You may do so by adding a `mix_url` configuration option to your application's `config/app.php` configuration file:

```php
'mix_url' => env('MIX_ASSET_URL', null)
```

After configuring the Mix URL, The `mix` function will prefix the configured URL when generating URLs to assets:

```text
https://cdn.example.com/js/app.js?id=1964becbdd96414518cd
```

## Browsersync Reloading

[BrowserSync](https://browsersync.io/) can automatically monitor your files for changes, and inject your changes into the browser without requiring a manual refresh. You may enable support for this by calling the `mix.browserSync()` method:

```javascript
mix.browserSync('laravel.test');
```

[BrowserSync options](https://browsersync.io/docs/options) may be specified by passing a JavaScript object to the `browserSync` method:

```javascript
mix.browserSync({
    proxy: 'laravel.test'
});
```

Next, start webpack's development server using the `npm run watch` command. Now, when you modify a script or PHP file you can watch as the browser instantly refreshes the page to reflect your changes.

## Environment Variables

You may inject environment variables into your `webpack.mix.js` script by prefixing one of the environment variables in your `.env` file with `MIX_`:

```php
MIX_SENTRY_DSN_PUBLIC=http://example.com
```

After the variable has been defined in your `.env` file, you may access it via the `process.env` object. However, you will need to restart the task if the environment variable's value changes while the task is running:

```javascript
process.env.MIX_SENTRY_DSN_PUBLIC
```

## Notifications

When available, Mix will automatically display OS notifications when compiling, giving you instant feedback as to whether the compilation was successful or not. However, there may be instances when you would prefer to disable these notifications. One such example might be triggering Mix on your production server. Notifications may be deactivated using the `disableNotifications` method:

```javascript
mix.disableNotifications();
```

