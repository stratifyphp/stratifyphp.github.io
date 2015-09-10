---
layout: documentation
---

# Getting started

## Requirements

Stratify requires:

- **PHP 7**
- [Composer](https://getcomposer.org/doc/00-intro.md)
- [Puli](http://docs.puli.io/en/latest/installation.html#installing-the-puli-cli)

## Installation

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

**Note:** this error handler middleware is just an example for learning. Stratify ships with a complete [error handler module](error-handling.md) that will be introduced below (so keep on reading).

## Modules

A Stratify module boils down to one thing: **a configuration file**. With that configuration, a module is able to expose pre-configured objects (services, middlewares, …) and offer extension points on these objects.

*Note:* if you are confused between modules and middlewares, it's simple: a middleware is a class/function, a module is a whole package (containing a configuration + classes). A middleware will often be distributed in a module so that it comes with a pre-configuration.

### Example

Here is an example of a configuration for a very basic "Twig module":

```php
return [
    Twig_Environment::class => function ($container) {
        $twig = new Twig_Environment(new PuliTemplateLoader);
        $twig->setExtensions($container->get('twig.extensions'));
        return $twig;
    },
    'twig.extensions' => [],
];
```

In this example, the module configuration:

- defines how to create `Twig_Environment` so that Twig can be injected and used in the application
- exposes the `twig.extensions` array (empty by default) so that you can register extensions in your application's config. Here is an example on how you can use that extension point to register your custom Twig extension:
    ```php
    return [
        'twig.extensions' => add([
            get(MyTwigExtension::class),
        ]),
    ];
    ```

### Usage

To use a module, you need to install it with Composer and then register it in the application:

```php
$modules = [
    'module1',
    'module2',
    'module3',
];

$app = new Application($http, $modules);
```

### Error handling

As we mentioned above, Stratify offers a "error handling" module that contains an error handler middleware (as well as other utilities). Since it's already installed with the `stratify/framework` package let's register it:

```php
$modules = [
    'error-handler',
];

$http = ...;

$app = new Application($http, $modules);
```

Now that it's config is registered, we can use the middleware in our `$http` architecture:

```php
use Stratify\ErrorHandlerModule\ErrorHandlerMiddleware;

// ...

$http = pipe([
    ErrorHandlerMiddleware::class,

    router([
        // ...
    ]),
]);

$app = new Application($http, $modules);
```

With that new architecture, the `ErrorHandlerMiddleware` will be created and invoked before the router so that it can catch any exception that happens in middlewares registered after it.

Most of the time you will want to have the error handler middleware run as the first middleware, so that it can catch all exceptions.
