# How to code a simple API in Laravel

## General Information

The Laravel API server tries to solve the scaffolding of an authentication system by reducing the amount of time you spend on developing these solutions over and over, eventually giving you a full-feature environment you can already start coding. The authentication is based on JWT tokens, making the project suitable for any frontend or mobile application and deployable anywhere.

## Decide the codebase structure

The codebase is the same as [Laravel's official directory structure](https://laravel.com/docs/8.x/structure). It's recommended to follow it to keep the app clean and properly managed.

## Libraries and modules

Beside the default Composer packages that Laravel comes with on each fresh installation, the following packages are used:

- [tymon/jwt-auth](https://packagist.org/packages/tymon/jwt-auth) - used to create JWT tokens

## Coding the routes

When coding the routes, keep in mind that the app is an API. You should use the `routes/api.php` file to define the routes for your API. Optionally, you may attach the API middleware to make sure the requests are coming from logged in users:

```php
Route::middleware(['auth:api'])->group(function () {
    //
});
```

Each default route comes with a controller, but you can use a controller for multiple routes:

```bash
php artisan make:controller MyController
```

```php
class MyController
{
    public function index()
    {
        return response()->json([
            'currentView' => 'index',
        ]);
    }

    public function show()
    {
        return response()->json([
            'currentView' => 'show',
        ]);
    }
}

Route::get('/my-path/index', [MyController::class, 'index']);
Route::get('/my-path/show', [MyController::class, 'show']);
```

For further reading, Laravel has an entire documentation about [Routing](https://laravel.com/docs/8.x/routing) that you can check on.

## Tests

The tests can be found into the `tests/` folder. We mainly use Feature tests to cover end-to-end scenarios instead of performing unit testing. With Feature testing, testing is done in a linear manner to test different scenarios like login - edit user - logout, which are much closer to reality.

The tests are modular, are segmented into files that define functionalities and each test describes well what's tested.

The tests are ran with PHPUnit:

```bash
vendor/bin/phpunit
```

## Docker

To install the project in Docker, you may use the following Dockerfile example. This Dockerfile is suitable for PHP-FPM, so you would need an NGINX container for it to work:

```dockerfile
FROM php:8.0-fpm-alpine

RUN apk --update add \
        wget \
        curl \
        build-base \
        composer \
        nodejs \
        npm \
        libmcrypt-dev \
        libxml2-dev \
        pcre-dev \
        zlib-dev \
        autoconf \
        oniguruma-dev \
        openssl \
        openssl-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        jpeg-dev \
        libpng-dev \
        imagemagick-dev \
        imagemagick \
        postgresql-dev \
        libzip-dev \
        gettext-dev \
        libxslt-dev \
        libgcrypt-dev && \
    pecl channel-update pecl.php.net && \
    pecl install mcrypt redis-5.3.4 && \
    docker-php-ext-install \
        mysqli \
        mbstring \
        pdo \
        pdo_mysql \
        tokenizer \
        xml \
        pcntl \
        bcmath \
        pdo_pgsql \
        zip \
        intl \
        gettext \
        soap \
        sockets \
        xsl && \
    docker-php-ext-configure gd --with-freetype=/usr/lib/ --with-jpeg=/usr/lib/ && \
    docker-php-ext-install gd && \
    docker-php-ext-enable redis && \
    rm -rf /tmp/pear && \
    rm /var/cache/apk/*

WORKDIR /var/www/html
```

---
Simple API Server - Laravel
