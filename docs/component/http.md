---
layout: documentation
---

# The Http component

The Http component is a non-opinionated package to write an HTTP application. Unlike the whole Stratify framework, it provides only the generic core of an HTTP application. It makes it good for:

- building very fast and lightweight micro-services or micro-applications
- building your application with your own architecture
- building a framework on top of it

## Concept

The Http package relies on a single concept: **middlewares**. An HTTP application runs by simply **calling a middleware**.

```php
$app = new Application($middleware);
$app->run();
```

The simplest middleware could look like this:

```php
$middleware = function (ServerRequestInterface $request, ResponseInterface $response, callable $next) {
    $response->getBody()->write('Hello world!');
    return $response;
}
$app = new Application($middleware);
$app->run();
```

You will note that such an application doesn't contain any router. The application is extremely simple and makes no assertion for you. If you don't need a router, you can avoid the overhead of it.

You could also consider implementing your own very specific router (and optimize it to your heart's content):

```php
$middleware = function (ServerRequestInterface $request, ResponseInterface $response, callable $next) {
    switch ($request->getRequestTarget()) {
        case '/':
            $response->getBody()->write('Home');
            break;
        case '/deploy':
            $response->getBody()->write('Deploying');
            break;
        default:
            return $next($request, $response);
    }
    return $response;
};
```

As you can see, a lot of freedom and no overhead.

## Composing middlewares

Writing an application with one callable can work for very specific micro-services, but can be limited. If you want to create a more complex application, you can take full advantages of middlewares by piping them.

```php
$middleware = new MiddlewarePipe([

    function ($request, $response, $next) {
        $res->getBody()->write('Hello');
        return $next($req, $res);        // call the next middleware
    },

    function ($request, $response, $next) {
        $res->getBody()->write(' world');
        return $res;
    },

]);

$app = new Application($middleware);
$app->run();
// The application will always return "Hello world" as a response
```

Each middleware in the pipe must return a response. To do this, it can either:

- build the response
- call the next middleware in the pipe and return the result
- call the next middleware in the pipe, play with the response and return it

That allows you to add any behavior you want **before or after** the request is processed by another middleware. You can also intercept and stop the application flow to reach the next middleware.

For example, you may want to use an authentication middleware to prevent un-authorized access to your application.
