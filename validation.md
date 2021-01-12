# Валидация



## Introduction

Laravel provides several different approaches to validate your application's incoming data. It is most common to use the `validate` method available on all incoming HTTP requests. However, we will discuss other approaches to validation as well.

Laravel includes a wide variety of convenient validation rules that you may apply to data, even providing the ability to validate if values are unique in a given database table. We'll cover each of these validation rules in detail so that you are familiar with all of Laravel's validation features.

## Validation Quickstart

To learn about Laravel's powerful validation features, let's look at a complete example of validating a form and displaying the error messages back to the user. By reading this high-level overview, you'll be able to gain a good general understanding of how to validate incoming request data using Laravel:

### Defining The Routes

First, let's assume we have the following routes defined in our `routes/web.php` file:

```php
use App\Http\Controllers\PostController;

Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

The `GET` route will display a form for the user to create a new blog post, while the `POST` route will store the new blog post in the database.

### Creating The Controller

Next, let's take a look at a simple controller that handles incoming requests to these routes. We'll leave the `store` method empty for now:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Show the form to create a new blog post.
     *
     * @return \Illuminate\View\View
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * Store a new blog post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // Validate and store the blog post...
    }
}
```

### Writing The Validation Logic

Now we are ready to fill in our `store` method with the logic to validate the new blog post. To do this, we will use the `validate` method provided by the `Illuminate\Http\Request` object. If the validation rules pass, your code will keep executing normally; however, if validation fails, an exception will be thrown and the proper error response will automatically be sent back to the user.

If validation fails during a traditional HTTP request, a redirect response to the previous URL will be generated. If the incoming request is an XHR request, a JSON response containing the validation error messages will be returned.

To get a better understanding of the `validate` method, let's jump back into the `store` method:

```php
/**
 * Store a new blog post.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // The blog post is valid...
}
```

As you can see, the validation rules are passed into the `validate` method. Don't worry - all available validation rules are [documented](https://laravel.com/docs/8.x/validation#available-validation-rules). Again, if the validation fails, the proper response will automatically be generated. If the validation passes, our controller will continue executing normally.

Alternatively, validation rules may be specified as arrays of rules instead of a single `|` delimited string:

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

In addition, you may use the `validateWithBag` method to validate a request and store any error messages within a [named error bag](https://laravel.com/docs/8.x/validation#named-error-bags):

```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

**Stopping On First Validation Failure**

Sometimes you may wish to stop running validation rules on an attribute after the first validation failure. To do so, assign the `bail` rule to the attribute:

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

In this example, if the `unique` rule on the `title` attribute fails, the `max` rule will not be checked. Rules will be validated in the order they are assigned.

**A Note On Nested Attributes**

If the incoming HTTP request contains "nested" field data, you may specify these fields in your validation rules using "dot" syntax:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

On the other hand, if your field name contains a literal period, you can explicitly prevent this from being interpreted as "dot" syntax by escaping the period with a backslash:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

### Отображение ошибок валидации <a id="displaying-the-validation-errors"></a>

So, what if the incoming request fields do not pass the given validation rules? As mentioned previously, Laravel will automatically redirect the user back to their previous location. In addition, all of the validation errors and [request input](https://laravel.com/docs/8.x/requests#retrieving-old-input) will automatically be [flashed to the session](https://laravel.com/docs/8.x/session#flash-data).

An `$errors` variable is shared with all of your application's views by the `Illuminate\View\Middleware\ShareErrorsFromSession` middleware, which is provided by the `web` middleware group. When this middleware is applied an `$errors` variable will always be available in your views, allowing you to conveniently assume the `$errors` variable is always defined and can be safely used. The `$errors` variable will be an instance of `Illuminate\Support\MessageBag`. For more information on working with this object, [check out its documentation](https://laravel.com/docs/8.x/validation#working-with-error-messages).

So, in our example, the user will be redirected to our controller's `create` method when validation fails, allowing us to display the error messages in the view:

```markup
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

**Customizing The Error Messages**

Laravel's built-in validation rules each have an error message that is located in your application's `resources/lang/en/validation.php` file. Within this file, you will find a translation entry for each validation rule. You are free to change or modify these messages based on the needs of your application.

In addition, you may copy this file to another translation language directory to translate the messages for your application's language. To learn more about Laravel localization, check out the complete [localization documentation](https://laravel.com/docs/8.x/localization).

**XHR Requests & Validation**

In this example, we used a traditional form to send data to the application. However, many applications receive XHR requests from a JavaScript powered frontend. When using the `validate` method during an XHR request, Laravel will not generate a redirect response. Instead, Laravel generates a JSON response containing all of the validation errors. This JSON response will be sent with a 422 HTTP status code.

**The @error Directive**

You may use the `@error` [Blade](https://laravel.com/docs/8.x/blade) directive to quickly determine if validation error messages exist for a given attribute. Within an `@error` directive, you may echo the `$message` variable to display the error message:

```markup
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title" type="text" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

### Repopulating Forms

When Laravel generates a redirect response due to a validation error, the framework will automatically [flash all of the request's input to the session](https://laravel.com/docs/8.x/session#flash-data). This is done so that you may conveniently access the input during the next request and repopulate the form that the user attempted to submit.

To retrieve flashed input from the previous request, invoke the `old` method on an instance of `Illuminate\Http\Request`. The `old` method will pull the previously flashed input data from the [session](https://laravel.com/docs/8.x/session):

```php
$title = $request->old('title');
```

Laravel also provides a global `old` helper. If you are displaying old input within a [Blade template](https://laravel.com/docs/8.x/blade), it is more convenient to use the `old` helper to repopulate the form. If no old input exists for the given field, `null` will be returned:

```markup
<input type="text" name="title" value="{{ old('title') }}">
```

### A Note On Optional Fields

By default, Laravel includes the `TrimStrings` and `ConvertEmptyStringsToNull` middleware in your application's global middleware stack. These middleware are listed in the stack by the `App\Http\Kernel` class. Because of this, you will often need to mark your "optional" request fields as `nullable` if you do not want the validator to consider `null` values as invalid. For example:

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

In this example, we are specifying that the `publish_at` field may be either `null` or a valid date representation. If the `nullable` modifier is not added to the rule definition, the validator would consider `null` an invalid date.

## Form Request Validation

### Creating Form Requests

For more complex validation scenarios, you may wish to create a "form request". Form requests are custom request classes that encapsulate their own validation and authorization logic. To create a form request class, you may use the `make:request` Artisan CLI command:

```bash
php artisan make:request StorePostRequest
```

The generated form request class will be placed in the `app/Http/Requests` directory. If this directory does not exist, it will be created when you run the `make:request` command. Each form request generated by Laravel has two methods: `authorize` and `rules`.

As you might have guessed, the `authorize` method is responsible for determining if the currently authenticated user can perform the action represented by the request, while the `rules` method returns the validation rules that should apply to the request's data:

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> You may type-hint any dependencies you require within the `rules` method's signature. They will automatically be resolved via the Laravel [service container](https://laravel.com/docs/8.x/container).

So, how are the validation rules evaluated? All you need to do is type-hint the request on your controller method. The incoming form request is validated before the controller method is called, meaning you do not need to clutter your controller with any validation logic:

```php
/**
 * Store a new blog post.
 *
 * @param  \App\Http\Requests\StorePostRequest  $request
 * @return Illuminate\Http\Response
 */
public function store(StorePostRequest $request)
{
    // The incoming request is valid...

    // Retrieve the validated input data...
    $validated = $request->validated();
}
```

If validation fails, a redirect response will be generated to send the user back to their previous location. The errors will also be flashed to the session so they are available for display. If the request was an XHR request, an HTTP response with a 422 status code will be returned to the user including a JSON representation of the validation errors.

**Adding After Hooks To Form Requests**

If you would like to add an "after" validation hook to a form request, you may use the `withValidator` method. This method receives the fully constructed validator, allowing you to call any of its methods before the validation rules are actually evaluated:

```php
/**
 * Configure the validator instance.
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```

### Authorizing Form Requests

The form request class also contains an `authorize` method. Within this method, you may determine if the authenticated user actually has the authority to update a given resource. For example, you may determine if a user actually owns a blog comment they are attempting to update. Most likely, you will interact with your [authorization gates and policies](https://laravel.com/docs/8.x/authorization) within this method:

```php
use App\Models\Comment;

/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

Since all form requests extend the base Laravel request class, we may use the `user` method to access the currently authenticated user. Also note the call to the `route` method in the example above. This method grants you access to the URI parameters defined on the route being called, such as the `{comment}` parameter in the example below:

```php
Route::post('/comment/{comment}');
```

If the `authorize` method returns `false`, an HTTP response with a 403 status code will automatically be returned and your controller method will not execute.

If you plan to handle authorization logic for the request in another part of your application, you may simply return `true` from the `authorize` method:

```php
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    return true;
}
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> You may type-hint any dependencies you need within the `authorize` method's signature. They will automatically be resolved via the Laravel [service container](https://laravel.com/docs/8.x/container).

### Customizing The Error Messages

You may customize the error messages used by the form request by overriding the `messages` method. This method should return an array of attribute / rule pairs and their corresponding error messages:

```php
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array
 */
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```

**Customizing The Validation Attributes**

Many of Laravel's built-in validation rule error messages contain an `:attribute` placeholder. If you would like the `:attribute` placeholder of your validation message to be replaced with a custom attribute name, you may specify the custom names by overriding the `attributes` method. This method should return an array of attribute / name pairs:

```php
/**
 * Get custom attributes for validator errors.
 *
 * @return array
 */
public function attributes()
{
    return [
        'email' => 'email address',
    ];
}
```

### Preparing Input For Validation

If you need to prepare or sanitize any data from the request before you apply your validation rules, you may use the `prepareForValidation` method:

```php
use Illuminate\Support\Str;

/**
 * Prepare the data for validation.
 *
 * @return void
 */
protected function prepareForValidation()
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

## Manually Creating Validators

If you do not want to use the `validate` method on the request, you may create a validator instance manually using the `Validator` [facade](https://laravel.com/docs/8.x/facades). The `make` method on the facade generates a new validator instance:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class PostController extends Controller
{
    /**
     * Store a new blog post.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // Store the blog post...
    }
}
```

The first argument passed to the `make` method is the data under validation. The second argument is an array of the validation rules that should be applied to the data.

After determining whether the request validation failed, you may use the `withErrors` method to flash the error messages to the session. When using this method, the `$errors` variable will automatically be shared with your views after redirection, allowing you to easily display them back to the user. The `withErrors` method accepts a validator, a `MessageBag`, or a PHP `array`.

### Automatic Redirection

If you would like to create a validator instance manually but still take advantage of the automatic redirection offered by the HTTP request's `validate` method, you may call the `validate` method on an existing validator instance. If validation fails, the user will automatically be redirected or, in the case of an XHR request, a JSON response will be returned:

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

You may use the `validateWithBag` method to store the error messages in a [named error bag](https://laravel.com/docs/8.x/validation#named-error-bags) if validation fails:

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

### Named Error Bags

If you have multiple forms on a single page, you may wish to name the `MessageBag` containing the validation errors, allowing you to retrieve the error messages for a specific form. To achieve this, pass a name as the second argument to `withErrors`:

```php
return redirect('register')->withErrors($validator, 'login');
```

You may then access the named `MessageBag` instance from the `$errors` variable:

```markup
{{ $errors->login->first('email') }}
```

### Customizing The Error Messages

If needed, you may use custom error messages for a validator instance instead of the default error messages. There are several ways to specify custom messages. First, you may pass the custom messages as the third argument to the `Validator::make` method:

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => 'The :attribute field is required.',
]);
```

In this example, the `:attribute` placeholder will be replaced by the actual name of the field under validation. You may also utilize other placeholders in validation messages. For example:

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

**Specifying A Custom Message For A Given Attribute**

Sometimes you may wish to specify a custom error message only for a specific attribute. You may do so using "dot" notation. Specify the attribute's name first, followed by the rule:

```php
$messages = [
    'email.required' => 'We need to know your email address!',
];
```

**Specifying Custom Attribute Values**

Many of Laravel's built-in error messages include an `:attribute:` placeholder that is replaced with the name of the field or attribute under validation. To customize the values used to replace these placeholders for specific fields, you may pass an array of custom attributes as the fourth argument to the `Validator::make` method:

```php
$validator = Validator::make($input, $rules, $messages, [
    'email' => 'email address',
]);
```

### After Validation Hook

You may also attach callbacks to be run after validation is completed. This allows you to easily perform further validation and even add more error messages to the message collection. To get started, call the `after` method on a validator instance:

```php
$validator = Validator::make(...);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add(
            'field', 'Something is wrong with this field!'
        );
    }
});

if ($validator->fails()) {
    //
}
```

## Working With Error Messages

After calling the `errors` method on a `Validator` instance, you will receive an `Illuminate\Support\MessageBag` instance, which has a variety of convenient methods for working with error messages. The `$errors` variable that is automatically made available to all views is also an instance of the `MessageBag` class.

**Retrieving The First Error Message For A Field**

To retrieve the first error message for a given field, use the `first` method:

```php
$errors = $validator->errors();

echo $errors->first('email');
```

**Retrieving All Error Messages For A Field**

If you need to retrieve an array of all the messages for a given field, use the `get` method:

```php
foreach ($errors->get('email') as $message) {
    //
}
```

If you are validating an array form field, you may retrieve all of the messages for each of the array elements using the `*` character:

```php
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

**Retrieving All Error Messages For All Fields**

To retrieve an array of all messages for all fields, use the `all` method:

```php
foreach ($errors->all() as $message) {
    //
}
```

**Determining If Messages Exist For A Field**

The `has` method may be used to determine if any error messages exist for a given field:

```php
if ($errors->has('email')) {
    //
}
```

### Specifying Custom Messages In Language Files

Laravel's built-in validation rules each have an error message that is located in your application's `resources/lang/en/validation.php` file. Within this file, you will find a translation entry for each validation rule. You are free to change or modify these messages based on the needs of your application.

In addition, you may copy this file to another translation language directory to translate the messages for your application's language. To learn more about Laravel localization, check out the complete [localization documentation](https://laravel.com/docs/8.x/localization).

**Custom Messages For Specific Attributes**

You may customize the error messages used for specified attribute and rule combinations within your application's validation language files. To do so, add your message customizations to the `custom` array of your application's `resources/lang/xx/validation.php` language file:

```php
'custom' => [
    'email' => [
        'required' => 'We need to know your email address!',
        'max' => 'Your email address is too long!'
    ],
],
```

### Specifying Attributes In Language Files

Many of Laravel's built-in error messages include an `:attribute:` placeholder that is replaced with the name of the field or attribute under validation. If you would like the `:attribute` portion of your validation message to be replaced with a custom value, you may specify the custom attribute name in the `attributes` array of your `resources/lang/xx/validation.php` language file:

```php
'attributes' => [
    'email' => 'email address',
],
```

### Specifying Values In Language Files

Some of Laravel's built-in validation rule error messages contain a `:value` placeholder that is replaced with the current value of the request attribute. However, you may occasionally need the `:value` portion of your validation message to be replaced with a custom representation of the value. For example, consider the following rule that specifies that a credit card number is required if the `payment_type` has a value of `cc`:

```php
Validator::make($request->all(), [
    'credit_card_number' => 'required_if:payment_type,cc'
]);
```

If this validation rule fails, it will produce the following error message:

```text
The credit card number field is required when payment type is cc.
```

Instead of displaying `cc` as the payment type value, you may specify a more user-friendly value representation in your `resources/lang/xx/validation.php` language file by defining a `values` array:

```php
'values' => [
    'payment_type' => [
        'cc' => 'credit card'
    ],
],
```

After defining this value, the validation rule will produce the following error message:

```text
The credit card number field is required when payment type is credit card.
```

## Available Validation Rules

Below is a list of all available validation rules and their function:

[Accepted](https://laravel.com/docs/8.x/validation#rule-accepted)[Active URL](https://laravel.com/docs/8.x/validation#rule-active-url)[After \(Date\)](https://laravel.com/docs/8.x/validation#rule-after)[After Or Equal \(Date\)](https://laravel.com/docs/8.x/validation#rule-after-or-equal)[Alpha](https://laravel.com/docs/8.x/validation#rule-alpha)[Alpha Dash](https://laravel.com/docs/8.x/validation#rule-alpha-dash)[Alpha Numeric](https://laravel.com/docs/8.x/validation#rule-alpha-num)[Array](https://laravel.com/docs/8.x/validation#rule-array)[Bail](https://laravel.com/docs/8.x/validation#rule-bail)[Before \(Date\)](https://laravel.com/docs/8.x/validation#rule-before)[Before Or Equal \(Date\)](https://laravel.com/docs/8.x/validation#rule-before-or-equal)[Between](https://laravel.com/docs/8.x/validation#rule-between)[Boolean](https://laravel.com/docs/8.x/validation#rule-boolean)[Confirmed](https://laravel.com/docs/8.x/validation#rule-confirmed)[Date](https://laravel.com/docs/8.x/validation#rule-date)[Date Equals](https://laravel.com/docs/8.x/validation#rule-date-equals)[Date Format](https://laravel.com/docs/8.x/validation#rule-date-format)[Different](https://laravel.com/docs/8.x/validation#rule-different)[Digits](https://laravel.com/docs/8.x/validation#rule-digits)[Digits Between](https://laravel.com/docs/8.x/validation#rule-digits-between)[Dimensions \(Image Files\)](https://laravel.com/docs/8.x/validation#rule-dimensions)[Distinct](https://laravel.com/docs/8.x/validation#rule-distinct)[Email](https://laravel.com/docs/8.x/validation#rule-email)[Ends With](https://laravel.com/docs/8.x/validation#rule-ends-with)[Exclude If](https://laravel.com/docs/8.x/validation#rule-exclude-if)[Exclude Unless](https://laravel.com/docs/8.x/validation#rule-exclude-unless)[Exists \(Database\)](https://laravel.com/docs/8.x/validation#rule-exists)[File](https://laravel.com/docs/8.x/validation#rule-file)[Filled](https://laravel.com/docs/8.x/validation#rule-filled)[Greater Than](https://laravel.com/docs/8.x/validation#rule-gt)[Greater Than Or Equal](https://laravel.com/docs/8.x/validation#rule-gte)[Image \(File\)](https://laravel.com/docs/8.x/validation#rule-image)[In](https://laravel.com/docs/8.x/validation#rule-in)[In Array](https://laravel.com/docs/8.x/validation#rule-in-array)[Integer](https://laravel.com/docs/8.x/validation#rule-integer)[IP Address](https://laravel.com/docs/8.x/validation#rule-ip)[JSON](https://laravel.com/docs/8.x/validation#rule-json)[Less Than](https://laravel.com/docs/8.x/validation#rule-lt)[Less Than Or Equal](https://laravel.com/docs/8.x/validation#rule-lte)[Max](https://laravel.com/docs/8.x/validation#rule-max)[MIME Types](https://laravel.com/docs/8.x/validation#rule-mimetypes)[MIME Type By File Extension](https://laravel.com/docs/8.x/validation#rule-mimes)[Min](https://laravel.com/docs/8.x/validation#rule-min)[Multiple Of](https://laravel.com/docs/8.x/validation#multiple-of)[Not In](https://laravel.com/docs/8.x/validation#rule-not-in)[Not Regex](https://laravel.com/docs/8.x/validation#rule-not-regex)[Nullable](https://laravel.com/docs/8.x/validation#rule-nullable)[Numeric](https://laravel.com/docs/8.x/validation#rule-numeric)[Password](https://laravel.com/docs/8.x/validation#rule-password)[Present](https://laravel.com/docs/8.x/validation#rule-present)[Regular Expression](https://laravel.com/docs/8.x/validation#rule-regex)[Required](https://laravel.com/docs/8.x/validation#rule-required)[Required If](https://laravel.com/docs/8.x/validation#rule-required-if)[Required Unless](https://laravel.com/docs/8.x/validation#rule-required-unless)[Required With](https://laravel.com/docs/8.x/validation#rule-required-with)[Required With All](https://laravel.com/docs/8.x/validation#rule-required-with-all)[Required Without](https://laravel.com/docs/8.x/validation#rule-required-without)[Required Without All](https://laravel.com/docs/8.x/validation#rule-required-without-all)[Same](https://laravel.com/docs/8.x/validation#rule-same)[Size](https://laravel.com/docs/8.x/validation#rule-size)[Sometimes](https://laravel.com/docs/8.x/validation#conditionally-adding-rules)[Starts With](https://laravel.com/docs/8.x/validation#rule-starts-with)[String](https://laravel.com/docs/8.x/validation#rule-string)[Timezone](https://laravel.com/docs/8.x/validation#rule-timezone)[Unique \(Database\)](https://laravel.com/docs/8.x/validation#rule-unique)[URL](https://laravel.com/docs/8.x/validation#rule-url)[UUID](https://laravel.com/docs/8.x/validation#rule-uuid)

**accepted**

The field under validation must be `"yes"`, `"on"`, `1`, or `true`. This is useful for validating "Terms of Service" acceptance or similar fields.

**active\_url**

The field under validation must have a valid A or AAAA record according to the `dns_get_record` PHP function. The hostname of the provided URL is extracted using the `parse_url` PHP function before being passed to `dns_get_record`.

**after:date**

The field under validation must be a value after a given date. The dates will be passed into the `strtotime` PHP function in order to be converted to a valid `DateTime` instance:

```php
'start_date' => 'required|date|after:tomorrow'
```

Instead of passing a date string to be evaluated by `strtotime`, you may specify another field to compare against the date:

```php
'finish_date' => 'required|date|after:start_date'
```

**after\_or\_equal:date**

The field under validation must be a value after or equal to the given date. For more information, see the [after](https://laravel.com/docs/8.x/validation#rule-after) rule.

**alpha**

The field under validation must be entirely alphabetic characters.

**alpha\_dash**

The field under validation may have alpha-numeric characters, as well as dashes and underscores.

**alpha\_num**

The field under validation must be entirely alpha-numeric characters.

**array**

The field under validation must be a PHP `array`.

**bail**

Stop running validation rules after the first validation failure.

**before:date**

The field under validation must be a value preceding the given date. The dates will be passed into the PHP `strtotime` function in order to be converted into a valid `DateTime` instance. In addition, like the [`after`](https://laravel.com/docs/8.x/validation#rule-after) rule, the name of another field under validation may be supplied as the value of `date`.

**before\_or\_equal:date**

The field under validation must be a value preceding or equal to the given date. The dates will be passed into the PHP `strtotime` function in order to be converted into a valid `DateTime` instance. In addition, like the [`after`](https://laravel.com/docs/8.x/validation#rule-after) rule, the name of another field under validation may be supplied as the value of `date`.

**between:min,max**

The field under validation must have a size between the given _min_ and _max_. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](https://laravel.com/docs/8.x/validation#rule-size) rule.

**boolean**

The field under validation must be able to be cast as a boolean. Accepted input are `true`, `false`, `1`, `0`, `"1"`, and `"0"`.

**confirmed**

The field under validation must have a matching field of `{field}_confirmation`. For example, if the field under validation is `password`, a matching `password_confirmation` field must be present in the input.

**date**

The field under validation must be a valid, non-relative date according to the `strtotime` PHP function.

**date\_equals:date**

The field under validation must be equal to the given date. The dates will be passed into the PHP `strtotime` function in order to be converted into a valid `DateTime` instance.

**date\_format:format**

The field under validation must match the given _format_. You should use **either** `date` or `date_format` when validating a field, not both. This validation rule supports all formats supported by PHP's [DateTime](https://www.php.net/manual/en/class.datetime.php) class.

**different:field**

The field under validation must have a different value than _field_.

**digits:value**

The field under validation must be _numeric_ and must have an exact length of _value_.

**digits\_between:min,max**

The field under validation must be _numeric_ and must have a length between the given _min_ and _max_.

**dimensions**

The file under validation must be an image meeting the dimension constraints as specified by the rule's parameters:

```php
'avatar' => 'dimensions:min_width=100,min_height=200'
```

Available constraints are: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_.

A _ratio_ constraint should be represented as width divided by height. This can be specified either by a fraction like `3/2` or a float like `1.5`:

```php
'avatar' => 'dimensions:ratio=3/2'
```

Since this rule requires several arguments, you may use the `Rule::dimensions` method to fluently construct the rule:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
    ],
]);
```

**distinct**

When validating arrays, the field under validation must not have any duplicate values:

```php
'foo.*.id' => 'distinct'
```

You may add `ignore_case` to the validation rule's arguments to make the rule ignore capitalization differences:

```php
'foo.*.id' => 'distinct:ignore_case'
```

**email**

The field under validation must be formatted as an email address. This validation rule utilizes the [`egulias/email-validator`](https://github.com/egulias/EmailValidator) package for validating the email address. By default the `RFCValidation` validator is applied, but you can apply other validation styles as well:

```php
'email' => 'email:rfc,dns'
```

The example above will apply the `RFCValidation` and `DNSCheckValidation` validations. Here's a full list of validation styles you can apply:

* `rfc`: `RFCValidation`
* `strict`: `NoRFCWarningsValidation`
* `dns`: `DNSCheckValidation`
* `spoof`: `SpoofCheckValidation`
* `filter`: `FilterEmailValidation`

The `filter` validator, which uses PHP's `filter_var` function, ships with Laravel and was Laravel's default email validation behavior prior to Laravel version 5.8.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> The `dns` and `spoof` validators require the PHP `intl` extension.

**ends\_with:foo,bar,...**

The field under validation must end with one of the given values.

**exclude\_if:anotherfield,value**

The field under validation will be excluded from the request data returned by the `validate` and `validated` methods if the _anotherfield_ field is equal to _value_.

**exclude\_unless:anotherfield,value**

The field under validation will be excluded from the request data returned by the `validate` and `validated` methods unless _anotherfield_'s field is equal to _value_.

**exists:table,column**

The field under validation must exist in a given database table.

**Basic Usage Of Exists Rule**

```php
'state' => 'exists:states'
```

If the `column` option is not specified, the field name will be used. So, in this case, the rule will validate that the `states` database table contains a record with a `state` column value matching the request's `state` attribute value.

**Specifying A Custom Column Name**

You may explicitly specify the database column name that should be used by the validation rule by placing it after the database table name:

```php
'state' => 'exists:states,abbreviation'
```

Occasionally, you may need to specify a specific database connection to be used for the `exists` query. You can accomplish this by prepending the connection name to the table name:

```php
'email' => 'exists:connection.staff,email'
```

Instead of specifying the table name directly, you may specify the Eloquent model which should be used to determine the table name:

```php
'user_id' => 'exists:App\Models\User,id'
```

If you would like to customize the query executed by the validation rule, you may use the `Rule` class to fluently define the rule. In this example, we'll also specify the validation rules as an array instead of using the `|` character to delimit them:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function ($query) {
            $query->where('account_id', 1);
        }),
    ],
]);
```

**file**

The field under validation must be a successfully uploaded file.

**filled**

The field under validation must not be empty when it is present.

**gt:field**

The field under validation must be greater than the given _field_. The two fields must be of the same type. Strings, numerics, arrays, and files are evaluated using the same conventions as the [`size`](https://laravel.com/docs/8.x/validation#rule-size) rule.

**gte:field**

The field under validation must be greater than or equal to the given _field_. The two fields must be of the same type. Strings, numerics, arrays, and files are evaluated using the same conventions as the [`size`](https://laravel.com/docs/8.x/validation#rule-size) rule.

**image**

The file under validation must be an image \(jpg, jpeg, png, bmp, gif, svg, or webp\).

**in:foo,bar,...**

The field under validation must be included in the given list of values. Since this rule often requires you to `implode` an array, the `Rule::in` method may be used to fluently construct the rule:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

**in\_array:anotherfield.\***

The field under validation must exist in _anotherfield_'s values.

**integer**

The field under validation must be an integer.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> This validation rule does not verify that the input is of the "integer" variable type, only that the input is a string or numeric value that contains an integer.

**ip**

The field under validation must be an IP address.

**ipv4**

The field under validation must be an IPv4 address.

**ipv6**

The field under validation must be an IPv6 address.

**json**

The field under validation must be a valid JSON string.

**lt:field**

The field under validation must be less than the given _field_. The two fields must be of the same type. Strings, numerics, arrays, and files are evaluated using the same conventions as the [`size`](https://laravel.com/docs/8.x/validation#rule-size) rule.

**lte:field**

The field under validation must be less than or equal to the given _field_. The two fields must be of the same type. Strings, numerics, arrays, and files are evaluated using the same conventions as the [`size`](https://laravel.com/docs/8.x/validation#rule-size) rule.

**max:value**

The field under validation must be less than or equal to a maximum _value_. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](https://laravel.com/docs/8.x/validation#rule-size) rule.

**mimetypes:text/plain,...**

The file under validation must match one of the given MIME types:

```php
'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'
```

To determine the MIME type of the uploaded file, the file's contents will be read and the framework will attempt to guess the MIME type, which may be different from the client's provided MIME type.

**mimes:foo,bar,...**

The file under validation must have a MIME type corresponding to one of the listed extensions.

**Basic Usage Of MIME Rule**

```php
'photo' => 'mimes:jpg,bmp,png'
```

Even though you only need to specify the extensions, this rule actually validates the MIME type of the file by reading the file's contents and guessing its MIME type. A full listing of MIME types and their corresponding extensions may be found at the following location:

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

**min:value**

The field under validation must have a minimum _value_. Strings, numerics, arrays, and files are evaluated in the same fashion as the [`size`](https://laravel.com/docs/8.x/validation#rule-size) rule.

**multiple\_of:value**

The field under validation must be a multiple of _value_.

**not\_in:foo,bar,...**

The field under validation must not be included in the given list of values. The `Rule::notIn` method may be used to fluently construct the rule:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'toppings' => [
        'required',
        Rule::notIn(['sprinkles', 'cherries']),
    ],
]);
```

**not\_regex:pattern**

The field under validation must not match the given regular expression.

Internally, this rule uses the PHP `preg_match` function. The pattern specified should obey the same formatting required by `preg_match` and thus also include valid delimiters. For example: `'email' => 'not_regex:/^.+$/i'`.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> When using the `regex` / `not_regex` patterns, it may be necessary to specify your validation rules using an array instead of using `|` delimiters, especially if the regular expression contains a `|` character.

**nullable**

The field under validation may be `null`.

**numeric**

The field under validation must be [numeric](https://www.php.net/manual/en/function.is-numeric.php).

**password**

The field under validation must match the authenticated user's password. You may specify an [authentication guard](https://laravel.com/docs/8.x/authentication) using the rule's first parameter:

```php
'password' => 'password:api'
```

**present**

The field under validation must be present in the input data but can be empty.

**regex:pattern**

The field under validation must match the given regular expression.

Internally, this rule uses the PHP `preg_match` function. The pattern specified should obey the same formatting required by `preg_match` and thus also include valid delimiters. For example: `'email' => 'regex:/^.+@.+$/i'`.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> When using the `regex` / `not_regex` patterns, it may be necessary to specify rules in an array instead of using `|` delimiters, especially if the regular expression contains a `|` character.

**required**

The field under validation must be present in the input data and not empty. A field is considered "empty" if one of the following conditions are true:

* The value is `null`.
* The value is an empty string.
* The value is an empty array or empty `Countable` object.
* The value is an uploaded file with no path.

**required\_if:anotherfield,value,...**

The field under validation must be present and not empty if the _anotherfield_ field is equal to any _value_.

If you would like to construct a more complex condition for the `required_if` rule, you may use the `Rule::requiredIf` method. This methods accepts a boolean or a closure. When passed a closure, the closure should return `true` or `false` to indicate if the field under validation is required:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf(function () use ($request) {
        return $request->user()->is_admin;
    }),
]);
```

**required\_unless:anotherfield,value,...**

The field under validation must be present and not empty unless the _anotherfield_ field is equal to any _value_.

**required\_with:foo,bar,...**

The field under validation must be present and not empty _only if_ any of the other specified fields are present.

**required\_with\_all:foo,bar,...**

The field under validation must be present and not empty _only if_ all of the other specified fields are present.

**required\_without:foo,bar,...**

The field under validation must be present and not empty _only when_ any of the other specified fields are not present.

**required\_without\_all:foo,bar,...**

The field under validation must be present and not empty _only when_ all of the other specified fields are not present.

**same:field**

The given _field_ must match the field under validation.

**size:value**

The field under validation must have a size matching the given _value_. For string data, _value_ corresponds to the number of characters. For numeric data, _value_ corresponds to a given integer value \(the attribute must also have the `numeric` or `integer` rule\). For an array, _size_ corresponds to the `count` of the array. For files, _size_ corresponds to the file size in kilobytes. Let's look at some examples:

```php
// Validate that a string is exactly 12 characters long...
'title' => 'size:12';

// Validate that a provided integer equals 10...
'seats' => 'integer|size:10';

// Validate that an array has exactly 5 elements...
'tags' => 'array|size:5';

// Validate that an uploaded file is exactly 512 kilobytes...
'image' => 'file|size:512';
```

**starts\_with:foo,bar,...**

The field under validation must start with one of the given values.

**string**

The field under validation must be a string. If you would like to allow the field to also be `null`, you should assign the `nullable` rule to the field.

**timezone**

The field under validation must be a valid timezone identifier according to the `timezone_identifiers_list` PHP function.

**unique:table,column,except,idColumn**

The field under validation must not exist within the given database table.

**Specifying A Custom Table / Column Name:**

Instead of specifying the table name directly, you may specify the Eloquent model which should be used to determine the table name:

```php
'email' => 'unique:App\Models\User,email_address'
```

The `column` option may be used to specify the field's corresponding database column. If the `column` option is not specified, the name of the field under validation will be used.

```php
'email' => 'unique:users,email_address'
```

**Specifying A Custom Database Connection**

Occasionally, you may need to set a custom connection for database queries made by the Validator. To accomplish this, you may prepend the connection name to the table name:

```php
'email' => 'unique:connection.users,email_address'
```

**Forcing A Unique Rule To Ignore A Given ID:**

Sometimes, you may wish to ignore a given ID during unique validation. For example, consider an "update profile" screen that includes the user's name, email address, and location. You will probably want to verify that the email address is unique. However, if the user only changes the name field and not the email field, you do not want a validation error to be thrown because the user is already the owner of the email address in question.

To instruct the validator to ignore the user's ID, we'll use the `Rule` class to fluently define the rule. In this example, we'll also specify the validation rules as an array instead of using the `|` character to delimit the rules:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> You should never pass any user controlled request input into the `ignore` method. Instead, you should only pass a system generated unique ID such as an auto-incrementing ID or UUID from an Eloquent model instance. Otherwise, your application will be vulnerable to an SQL injection attack.

Instead of passing the model key's value to the `ignore` method, you may also pass the entire model instance. Laravel will automatically extract the key from the model:

```php
Rule::unique('users')->ignore($user)
```

If your table uses a primary key column name other than `id`, you may specify the name of the column when calling the `ignore` method:

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

By default, the `unique` rule will check the uniqueness of the column matching the name of the attribute being validated. However, you may pass a different column name as the second argument to the `unique` method:

```php
Rule::unique('users', 'email_address')->ignore($user->id),
```

**Adding Additional Where Clauses:**

You may specify additional query conditions by customizing the query using the `where` method. For example, let's add a query condition that scopes the query to only search records that have an `account_id` column value of `1`:

```php
'email' => Rule::unique('users')->where(function ($query) {
    return $query->where('account_id', 1);
})
```

**url**

The field under validation must be a valid URL.

**uuid**

The field under validation must be a valid RFC 4122 \(version 1, 3, 4, or 5\) universally unique identifier \(UUID\).

## Conditionally Adding Rules

**Skipping Validation When Fields Have Certain Values**

You may occasionally wish to not validate a given field if another field has a given value. You may accomplish this using the `exclude_if` validation rule. In this example, the `appointment_date` and `doctor_name` fields will not be validated if the `has_appointment` field has a value of `false`:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, [
    'has_appointment' => 'required|bool',
    'appointment_date' => 'exclude_if:has_appointment,false|required|date',
    'doctor_name' => 'exclude_if:has_appointment,false|required|string',
]);
```

Alternatively, you may use the `exclude_unless` rule to not validate a given field unless another field has a given value:

```php
$validator = Validator::make($data, [
    'has_appointment' => 'required|bool',
    'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
    'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
]);
```

**Validating When Present**

In some situations, you may wish to run validation checks against a field **only** if that field is present in the data being validated. To quickly accomplish this, add the `sometimes` rule to your rule list:

```php
$v = Validator::make($request->all(), [
    'email' => 'sometimes|required|email',
]);
```

In the example above, the `email` field will only be validated if it is present in the `$data` array.

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> If you are attempting to validate a field that should always be present but may be empty, check out [this note on optional fields](https://laravel.com/docs/8.x/validation#a-note-on-optional-fields)

**Complex Conditional Validation**

Sometimes you may wish to add validation rules based on more complex conditional logic. For example, you may wish to require a given field only if another field has a greater value than 100. Or, you may need two fields to have a given value only when another field is present. Adding these validation rules doesn't have to be a pain. First, create a `Validator` instance with your _static rules_ that never change:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```

Let's assume our web application is for game collectors. If a game collector registers with our application and they own more than 100 games, we want them to explain why they own so many games. For example, perhaps they run a game resale shop, or maybe they just enjoy collecting games. To conditionally add this requirement, we can use the `sometimes` method on the `Validator` instance.

```php
$v->sometimes('reason', 'required|max:500', function ($input) {
    return $input->games >= 100;
});
```

The first argument passed to the `sometimes` method is the name of the field we are conditionally validating. The second argument is a list of the rules we want to add. If the closure passed as the third argument returns `true`, the rules will be added. This method makes it a breeze to build complex conditional validations. You may even add conditional validations for several fields at once:

```php
$v->sometimes(['reason', 'cost'], 'required', function ($input) {
    return $input->games >= 100;
});
```

> ![](https://laravel.com/img/callouts/lightbulb.min.svg)
>
> The `$input` parameter passed to your closure will be an instance of `Illuminate\Support\Fluent` and may be used to access your input and files under validation.

## Validating Arrays

Validating array based form input fields doesn't have to be a pain. You may use "dot notation" to validate attributes within an array. For example, if the incoming HTTP request contains a `photos[profile]` field, you may validate it like so:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

You may also validate each element of an array. For example, to validate that each email in a given array input field is unique, you may do the following:

```php
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

Likewise, you may use the `*` character when specifying [custom validation messages in your language files](https://laravel.com/docs/8.x/validation#custom-messages-for-specific-attributes), making it a breeze to use a single validation message for array based fields:

```php
'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must have a unique email address',
    ]
],
```

## Custom Validation Rules

### Using Rule Objects

Laravel provides a variety of helpful validation rules; however, you may wish to specify some of your own. One method of registering custom validation rules is using rule objects. To generate a new rule object, you may use the `make:rule` Artisan command. Let's use this command to generate a rule that verifies a string is uppercase. Laravel will place the new rule in the `app/Rules` directory. If this directory does not exist, Laravel will create it when you execute the Artisan command to create your rule:

```bash
php artisan make:rule Uppercase
```

Once the rule has been created, we are ready to define its behavior. A rule object contains two methods: `passes` and `message`. The `passes` method receives the attribute value and name, and should return `true` or `false` depending on whether the attribute value is valid or not. The `message` method should return the validation error message that should be used when validation fails:

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class Uppercase implements Rule
{
    /**
     * Determine if the validation rule passes.
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        return strtoupper($value) === $value;
    }

    /**
     * Get the validation error message.
     *
     * @return string
     */
    public function message()
    {
        return 'The :attribute must be uppercase.';
    }
}
```

You may call the `trans` helper from your `message` method if you would like to return an error message from your translation files:

```php
/**
 * Get the validation error message.
 *
 * @return string
 */
public function message()
{
    return trans('validation.uppercase');
}
```

Once the rule has been defined, you may attach it to a validator by passing an instance of the rule object with your other validation rules:

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

### Using Closures

If you only need the functionality of a custom rule once throughout your application, you may use a closure instead of a rule object. The closure receives the attribute's name, the attribute's value, and a `$fail` callback that should be called if validation fails:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function ($attribute, $value, $fail) {
            if ($value === 'foo') {
                $fail('The '.$attribute.' is invalid.');
            }
        },
    ],
]);
```

### Implicit Rules

By default, when an attribute being validated is not present or contains an empty string, normal validation rules, including custom rules, are not run. For example, the [`unique`](https://laravel.com/docs/8.x/validation#rule-unique) rule will not be run against an empty string:

```php
use Illuminate\Support\Facades\Validator;

$rules = ['name' => 'unique:users,name'];

$input = ['name' => ''];

Validator::make($input, $rules)->passes(); // true
```

For a custom rule to run even when an attribute is empty, the rule must imply that the attribute is required. To create an "implicit" rule, implement the `Illuminate\Contracts\Validation\ImplicitRule` interface. This interface serves as a "marker interface" for the validator; therefore, it does not contain any additional methods you need to implement beyond the methods required by the typical `Rule` interface.

> ![](https://laravel.com/img/callouts/exclamation.min.svg)
>
> An "implicit" rule only _implies_ that the attribute is required. Whether it actually invalidates a missing or empty attribute is up to you.

