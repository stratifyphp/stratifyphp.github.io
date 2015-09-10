---
layout: documentation
---

# Getting started

Please note that Stratify requires **PHP 7**, [Composer](https://getcomposer.org/doc/00-intro.md) and [Puli](http://docs.puli.io/en/latest/installation.html#installing-the-puli-cli).

## Installation

To create a new project using Stratify, start with the following `composer.json`:

```json
{
    "require": {
        "stratify/framework": "@dev"
    },
    "minimum-stability": "beta"
}
```

The "beta" minimum stability is needed because Puli is not stable yet.

Now run:

```
composer install
```

## Hello world

Now that the framework is installed, it is up to you to create your application's architecture. To help you get started, let's write a very simple application.

Create a `index.php` file that will receive all HTTP requests for your application:

```php
<?php

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

require __DIR__ . '/vendor/autoload.php';

$http = function (ServerRequestInterface $request, ResponseInterface $response, callable $next) {
    $response->getBody()->write('Hello world!');
    return $response;
};

$app = new Stratify\Http\Application($http);
$app->run();
```

As you can guess, this application will simply display "Hello world!". The application in itself is very simple: it is composed of **one middleware** whose role is to return a response.

A middleware is a callable that takes PSR-7 request and response object and returns a response. It also takes a "next" middleware to call in case the current middleware cannot provide a response.

**The key in creating your application is to compose middlewares.** For example you can replace the application's middleware by a *middleware pipe*, a router, etc. Let's move on the next section to go further.

## Micro-application

While the previous "Hello world" didn't look very useful, it will show you one core feature of Stratify: scalability. You are able to scale from a nano-application to a very complex one very simply.

To create a useful micro-application, let's replace the core middleware by **a router**:

```php
use function Stratify\Framework\router;

require __DIR__ . '/vendor/autoload.php';

$http = router([
    '/' => function () {
        return 'Welcome to the home page.';
    },
    '/number/{number}' => function ($number) {
        return 'You have asked for number ' . $number;
    },
]);

$app = new Stratify\Http\Application($http);
$app->run();
```

2 things to note:

- the `router()` function is a helper to create a pre-configured [Router](../component/router.md) instance
- router controllers are middlewares, but with (optional) sugar to write them with pleasure

To learn all about routing in the framework, read the [routing](routing.md) guide.

## Composing your application with middlewares

As it is, this micro-application does the job but misses a basic feature: error handling. Let's add a (very basic) error handler to demonstrate how to compose middlewares in a more advanced way.

```php
use function Stratify\Framework\router;
use function Stratify\Framework\pipe;

// ...

$http = pipe([

    // Error handling
    function ($request, $response, $next) {
        try {
            return $next($request, $response);
        } catch (\Exception $e) {
            $message = 'An error occurred: '.$e->getMessage();
            $response->getBody()->write($message);
            return $response->withStatus(500); // HTTP status 500
        }
    },

    router([
        // ...
    ]),

]);
```

We created a **middleware pipe**: each middleware is executed in sequence until one returns a response. That allows to intercept, replace or decorate the next middlewares.

For our example, our error handler runs the next middleware (the router) in a `try/catch`. In case of an exception, it will catch it and return an error response.

**Note:** this error handler middleware is just an example for learning. Stratify ships with a complete [error handler module](error-handling.md) that will be introduced below (so keep on reading).

## Modules

Let's get to the next level and make use of [Stratify modules](modules.md). Modules are reusable packages that can provide you PHP classes (middlewares, libraries, …), resources (assets, views, …) and configuration.

Instead of using our basic error handler shown above, we will instead use Stratify's error handler module. This module provides a middleware that will render a nice error page in case of an exception.

Since it's already installed with the `stratify/framework` package let's directly register it:

```php
$modules = [
    'error-handler',
];

$http = ...;

$app = new Application($http, $modules);
```

Registering a module means registering its configuration. So now that it's configuration is registered, we can use the "error handler" middleware in our `$http` architecture:

```php
use Stratify\ErrorHandlerModule\ErrorHandlerMiddleware;

$modules = [
    'error-handler',
];

$http = pipe([
    ErrorHandlerMiddleware::class,

    router([
        // ...
    ]),
]);

$app = new Application($http, $modules);
```

With that new architecture, the `ErrorHandlerMiddleware` will be created and invoked before the router so that it can catch any exception that happens in middlewares registered after it.

TODO: `app` module

## What's next

- learn about [Modules](modules.md)
