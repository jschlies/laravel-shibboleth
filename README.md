# Laravel Shibboleth Service Provider
Forked from [razorbacks/laravel-shibboleth](https://github.com/razorbacks/ubuntu-authentication/tree/master/shibboleth) for use at The University of Illinois at Chicago. This package provides Shibboleth authentication for Laravel at UIC.

[![Code Coverage][8]][7]

## Pre-Requisites

In order to use this plugin, we assume you already have a pre-existing
Shibboleth SP and Shibboleth IdP configured. This does not (and will not) go
into explaining how to set that up.

## Installation

Use [composer][1] to require the latest release into your project:

    composer require uicosss/laravel-shibboleth

If you're running Laravel >= 5.5, then you can skip this step, otherwise
you will need to manually register the service provider in your `config/app.php`
file within the `Providers` array.

```php
StudentSystemServices\Shibboleth\ShibbolethServiceProvider::class,
```

If you would like to use the emulated IdP via shibalike, then you will need
to manually register it on any version - this is not automatically loaded even
in Laravel 5.5.

```php
StudentSystemServices\Shibboleth\ShibalikeServiceProvider::class,
```

Publish the default configuration file:

    php artisan vendor:publish --provider="StudentSystemServices\Shibboleth\ShibbolethServiceProvider"

Change the driver to `shibboleth` in your `config/auth.php` file.

```php
'providers' => [
    'users' => [
        'driver' => 'shibboleth',
        'model'  => App\User::class,
    ],
],
```

Now users may login via Shibboleth by going to `https://example.uic.edu/shibboleth-login`
and logout using `https://example.uic.edu/shibboleth-logout` so you can provide a custom link
or redirect based on email address in the login form.

```php
@if (Auth::guest())
    <a href="/shibboleth-login">Login</a>
@else
    <a href="/shibboleth-logout">
        Logout {{ Auth::user()->name }}
    </a>
@endif
```

You may configure server variable mappings in `config/shibboleth.php` such as
the user's first name, last name, entitlements, etc. You can take a look at them
by reading what's been populated into the `$_SERVER` variable after authentication.

```php
<?php print_r($_SERVER);
```

Mapped values will be synced to the user table upon successful authentication.

### Declare Login Route

By convention, [laravel assumes a route named `login` exists][laravel-login]
to redirect unauthenticated requests.

[laravel-login]:https://github.com/laravel/framework/blob/b301be35ca42762d876f84d02704718a1226a97d/src/Illuminate/Foundation/Exceptions/Handler.php#L220

This package names its route `shibboleth-login` because
it's designed to work alongside other authentication providers,
such as the default scaffolding provided by artisan.
But if this is the only authentication provider,
then that name will need to be manually declared. e.g.

```php
Route::name('login')->get('/login', '\\'.Route::getRoutes()->getByName('shibboleth-login')->getActionName());
```

or more readable, but with a redirect:

```php
Route::redirect('/login', '/shibboleth-login')->name('login');
```

## Local Users

This was designed to work side-by-side with the native authentication system
for projects where you want to have both Shibboleth and local users.
If you would like to allow local registration as well as authenticate Shibboleth
users, then use laravel's built-in auth system.

    php artisan make:auth


[1]:https://getcomposer.org/
[2]:https://codeclimate.com/github/razorbacks/laravel-shibboleth
[3]:https://codeclimate.com/github/razorbacks/laravel-shibboleth/badges/gpa.svg
[4]:https://github.com/tymondesigns/jwt-auth
[7]:https://codecov.io/gh/razorbacks/laravel-shibboleth/branch/master
[8]:https://img.shields.io/codecov/c/github/razorbacks/laravel-shibboleth/master.svg
[11]:https://travis-ci.org/razorbacks/laravel-shibboleth
[12]:https://travis-ci.org/razorbacks/laravel-shibboleth.svg?branch=master
[13]:https://github.com/mrclay/shibalike
[14]:https://laravel.com/docs/5.4/eloquent-relationships#many-to-many
[15]:./src/database/migrations/2017_02_24_100000_create_entitlement_user_table.php
[16]:https://laravel.com/docs/5.4/authorization
