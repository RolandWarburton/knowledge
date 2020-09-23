# Laravel Bootstrap Notes

## Useful Extensions

* **kokororin.vscode-phpfmt** - PHP Formatting.
* **onecentlin.laravel-blade** - Laravel Blade Snippets.
* **felixbecker.php-intellisense** - PHP Intellisense.

## Get composer

```none
mkdir ~/composer && cd ~/composer && \
curl -sS https://getcomposer.org/installer -o composer-setup.php && \
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

Now install laravel installer globally.

```none
composer global require laravel/installer
```

Now add composer to the bin. The laravel binary could either be in `~/.composer` or `~/.config/.composer`.

```none
# Check that composer exists here
ls ~/.config/composer/vendor/bin
```

Then add it to `~/.profile` for whichever shell you are using.

```sh
if [ -d "$HOME/.config/composer/vendor/bin" ] ; then
    PATH="$HOME/.config/composer/vendor/bin:$PATH"
fi
```

this will only work after you restart your shell so to make the change work right away export it into your path with `export PATH="$HOME/.config/composer/vendor/bin:$PATH"`.

## Creating a laravel project

Get going with.

```none
laravel new myproject
```

```output
| |                             | |
| |     __ _ _ __ __ ___   _____| |
| |    / _` | '__/ _` \ \ / / _ \ |
| |___| (_| | | | (_| |\ V /  __/ |
|______\__,_|_|  \__,_| \_/ \___|_|

Creating a "laravel/laravel" project at "./myproject"
Installing laravel/laravel (v8.0.1)

<output omitted>
```

If you run into an error thats something like `Your requirements could not be resolved to an installable set of packages.`. Simply google for these packages and install them, in my case i was missing the package phpunit/phpunit on debian 10, i resolved it with `sudo apt install php-xml`.

Great! Your project is now bootstrapped. One last thing to mention is that you should add `vendor` to your `.gitignore` to stop tracking the library code.

## Artisan

PHP artisan is a set of command line tools for managing the laravel framework.

```none
php artisan
```

```output
Laravel Framework 8.1.0
Usage:
  command [options] [arguments]

Available commands:
clear-compiled       Remove the compiled class file
down                 Put the application into maintenance / demo mode
env                  Display the current framework environment
help                 Displays help for a command
inspire              Display an inspiring quote
list                 Lists commands
migrate              Run the database migrations
optimize             Cache the framework bootstrap files
serve                Serve the application on the PHP development server
test                 Run the application tests
tinker               Interact with your application
up                   Bring the application out of maintenance mode

<output omitted>
```

#### Start the development server with artisan

Run the php dev server.

```none
php artisan serve
```

```output
Starting Laravel development server: http://127.0.0.1:8000
```

I run artisan on a dev server so to make it accessible on the LAN add the `--host=10.10.10.12` flag. You can also change the port with the `--port=8000` flag. An even better way is just to open up all addresses with `--host=0.0.0.0`.

#### Resolving 500 error after server start

I ran into an issue immediately when navigating to the website and seeing a **500 Internal Error** message, this was caused by no .env or .env.sample being generated. I resolved this by copying the sample from the laravel github [here](https://github.com/laravel/laravel/blob/master/.env.example). I pasted it below for convenience, this is for laravel installed version 4.0.3, artisan version 8.1.0 (`php artisan --version`).

```none
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

After Copying in the .env and creating a .env.sample as a template for later use you need to regenerate the keys and clear the cache (other wisdom from [laravel/framework 9080](https://github.com/laravel/framework/issues/9080) suggest using `config:cache` instead of `config:clear`, however the latter seems to have worked for me).

Go ahead and run this to refresh your keys, then reload and you should have solved the 500 internal error issue.

```none
php artisan key:generate
php artisan config:clear
```

## Observing the views

Laravel uses a view engine called blade, its similar to EJS in node as it allows injection of code (php) into a "html" template (.blade.php).

Views are stored in `resources/views`.

<!-- ## Authentication

```none
php artisan make:auth
``` -->

## Routing

Routes are defined in `routes/web.php`. Each route uses a blade template located in `resources/views`. When you define a route in web.php you specify the template relative to the views directory.

```php
// Simple route
Route::get('/', function () {
    return view('index');
});

// A route thats using a template in a nested directory
Route::get('/about', function () {
    return view('about/aboutTemplate');
});
```

### Partials

Partials work just how you would expect in other templating languages. Create a folder to house partials in `resources/views/partials`. Then create a blade partial, for example `nav.blade.php`.

```html
<!-- nav.blade.php -->
<nav>
	<div class="text-gray-500">
		<a href="/">home</a>
		<a href="/about">about</a>
		<a href="/contact">contact</a>
	</div>
</nav>
```

```php
// index.blade.php
// note no semicolon
@include("partials/nav")
```

### Blade routing

This addresses the issue where routes become complex and you can no longer hard code them. Instead you can use a blade template function `route()`. Lets modify the partial for that.

```php
// nav.blade.php
<nav>
	<div class="text-gray-500">
		<a href="/">home</a>
		<a href="{{route('about-page')}}">about</a>
		<a href="/contact">contact</a>
	</div>
</nav>
```

Next lets use method chaining to extend the route in index.php to chain it with a name. This way if */about* changes, the nav partial that relies on it wont. The effect of this is that now we can target the *route of the name "about-page", rather than the route of /about*. So *"about-page"* will always resolve to the correct route.

```php
Route::get('/about', function () {
    return view('about/about');
})->name('about-page');

```

## Components

Components are an extension of partials that allow for parameters to be parsed in.

Create another partial for a hero element in `resources/views/partials/hero.blade.php`, this time include a *slot*, this is the parameter and turns the partial into a **component**.

```php
// resources/views/partials/hero.blade.php
<div>
	<h1>Hero element</h1>
	{{ $slot }}
</div>
```

To use the component use the `@component` tag. For example in your main `resources/views/index.blade.php`.

```php
<!-- resources/views/index.blade.php -->
@component('partials/hero')
	This is an argument
@endcomponent
```

Will render.

```output
<div>
	<h1>Hero element</h1>
	This is an argument
</div>
```

## Laravel mix - bundling sass

Laravel mix is a wrapper for webpack, so we are going to create a style loader for sass through laravel mix.

Observe `/webpack.mix.js` and see its use of post-css as a build step for webpack, laravel mix handles the implementation of this behind the scenes and generates the actual webpack file for you. We will add the sass bundler later.

Next observe `resources/sass`, or create it if it doesn't exist. Then create an additional `app.scss`, `_variables.scss`, and `_reset.scss` etc...

Next ensure that you have the required node_modules that laravel requires, install these with `npm i`.

Next observe the avaliable scripts in `/package.json`. Run the dev script with `npm run dev`.

```output
roland@devel:~/blogBuilder/frontend$ npm run dev
<output emitted>
<expect additional packages to be installed on the first run>

DONE  Compiled successfully in 1779ms                                                                        8:17:41 PM

       Asset     Size   Chunks             Chunk Names
/css/app.css  0 bytes  /js/app  [emitted]  /js/app
/js/app.js  594 KiB  /js/app  [emitted]  /js/app
```

The important output here is that laravel mix has produced a css bundle using the css inside `resources/css/app.css` as per the configuration in `/webpack.mix.js`.

Lets add sass to the bundling options.

```js
// you can optionally keep postCss with no repercussions.
// However i think its better to remove it and rely just on the .sass
mix.js("resources/js/app.js", "public/js")
	.sass("resources/sass/app.scss", "public/css")
	.postCss("resources/css/app.css", "public/css", [
		//
	]);
```

Now re run `npm run dev` and observe node-sass being installed, then re run again to compile the targeted sass. You can see these changes bundled in `/public`. Note that by default `/resources/css/app.css` does not contain any styles so you wont be able to see any css being bundled, you can add some to test that its working correctly.

Finally link to the css dynamically using the `mix()` function provided by laravel.

```php
<link rel="stylesheet" href="{{mix('/css/app.css')}}">
```

Lastly observe the `npm run watch` command, this runs the dev command as a daemon and wont exit until told to, its using the webpack *--watch* flag on the backend but its not really important.

#### Bundle stuff with sass

Heres some tricks that you can use to make relying on sass much easier

```scss
// import google fonts from https://fonts.google.com
@import url("https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700&display=swap");
```

```scss
// after you have installed bootstrap you can require it
// use ~ to search relative to node_modules
@import "~bootstrap/scss/bootstrap";
```

#### Minify bundles

Minifying can be done with your favorite npm packages, such as CSSO for css and terser for js. Lets minify the css by using the inbuilt minify functionality built into laravel mix.

```js
mix.js("resources/js/app.js", "public/js")
	.sass("resources/sass/app.scss", "public/css")
	.minify("public/css/app.css");
```

This will generate an */css/app.min.css* so be sure to update your reference to it in *mix()*.

#### Laravel Mix post-build tasks

This can be done with the `mix.then()` promise.

```js
mix.then(() => {
	console.log("hey i will run after webpack finishes")
});
```

## Yield

<!-- TODO fill this out -->
`@yield` is a way of modularizing components and sharing them between pages easily. Its a good idea to abstract bits away from the actual 'index' and instead *extend* it like a class, treating index like a base.

Lets restructure the views directory

```output
resources/views/
├── pages
│   └── home.blade.php <- Create this
├── partials
│   ├── hero.blade.php
│   └── nav.blade.php
└── index.blade.php
```

Now lets edit `/resources/views/pages/home.blade.php`.

* We use the *extends* keyword to define which **base** we are extending functionality to (like class inheritance). We are extending the *index.blade.php* because its contents is generic.
* To create our yield-able component use a *section*, a section is a targeted "part" of a page.
* Inside this section put some content, in this case we include a partial and a component (see components above if this is not understood).

```php
// This is like the NAME of the base module
@extends('index')

// This section is used to yield is called by its name
@section('content')
	// include some example partial
	@include("partials/nav")
	// include some example component
	@component('partials/hero')
		This is an argument
	@endcomponent
@endsection
```

Now jump to `resources/views/index.blade.php` and just add yield, specifying which section you want to use.

```php
@yield('content')
```

**Home is now the new landing page.** Now you can think of *index.blade.php* as a **base** and *home.blade.php* as the actual component that we want to display, this means that *home* is the new landing page (home is going to be the new web root). Lets update the routing to reflect that in `/routes/web.php`.

```php
Route::get('/', function () {
	// Update the view to use pages/home to use our new component code
    return view('pages/home');
})->name("root");
```

#### Yield to extend more pages

Continuing on from extending *page* from *index*. Lets now do the about page by extending *about* from *index*.

```output
resources/views/
├── pages
│   ├── about.blade.php <- Create this
│   └── home.blade.php
├── partials
│   ├── hero.blade.php
│   └── nav.blade.php
└── index.blade.php
```

Then in the same style, copy the exact same content from `pages/home.blade.php` to `pages/about.blade.php`, you can change the name if you like to something meaningful for that page.

```php
@extends('index')

@section('content')
	@include("partials/nav")
	@component('partials/hero')
		About_Page
	@endcomponent
@endsection
```

And update the route in `/routes/web.php` to reflect the change in view location (`return return view("pages/about")`).

Now you have two pages, both extend index.html and each page can contain some unique text using the *partials/hero* component we wrote above.

## Controllers

Lets get started on the controller portion of MVC. Observe `/app/Http/controllers` to see where controllers are placed, by default there is a default controller called *Controller.php*. Generate a new controller with `php artisan make:controller FrontEndController` to create a controller for the front end routing.

Now lets add some functionality to that controller.

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class FrontEndController extends Controller
{
	public function home () {
		return view("pages/home");
	}
}
```

Now modify the routes to use this as a handler. FrontEndController is the class name and home is the method name. **IN OLD LARAVEL VERSIONS (<=7>)** the RouteServiceProvider had its namespace removed. To fix this you would have to use the Fully Qualified Class Name for your Controllers when referring to them in your routes when not using the namespace prefixing. [source](https://stackoverflow.com/questions/63807930/target-class-controller-does-not-exist-laravel-8).

```php
// "FrontEndController" matching the controller we made
// "home" matching the public function we put in "FrontEndController"
Route::get('/', [FrontEndController::class, 'home'])->name("root");
```

## Bulma

Install bulma.

```none
npm i bulma
```

Then import modular components from node_modules/bulma using the `~` prepend in sass files. **Make sure to include ~bulma/sass/utilities/_all**.

```scss
// example importing buttons only
@import "~bulma/sass/utilities/_all";
@import "~bulma/sass/elements/button";
```

```html
<button class="button is-primary is-green">Boop the button</button>
```
