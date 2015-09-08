---
layout: documentation
---

# Getting started

## Installation

Please note Stratify requires **PHP 7**.

When creating a new project using Stratify, you can start by requiring the framework:

```
composer require stratify/framework
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

- the `router()` function is a helper to create a preconfigured [Router](../component/router.md) instance
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

For our example, our error handler runs the next middleware (the rotuer) in a `try/catch`. In case of an exception, it will catch it and return an error response.

**Note:** this error handler middleware is just an example for learning. Stratify ships with a complete [error handler module](error-handling.md) that will be introduced in the next paragraph (so keep on reading).

## Modules


