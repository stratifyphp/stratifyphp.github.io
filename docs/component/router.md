---
layout: documentation
---

# The Router component

The Router component provides several routers **implemented as middlewares**.

## The classic router

`Stratify\Router\Router` (aka the "classic" router) routes HTTP requests to handlers.

Example:

```php
$router = new Router([
    '/' => function ($request, $response, $next) {
        $response->getBody()->write('Hello world!');
        return $response;
    },
]);
```

### Routes

Routes are defined by mapping a URI path to a controller. By default only HTTP `GET` requests are allowed for a route.

If you need to add more constraints to the route, you can use the `route()` function helper:

```php
use function Stratify\Router\route;

$router = new Router([
    '/' => route('controller')->method('POST'),
]);
```

The route helper allows to set the following constraints:

- HTTP method: `->method('GET', 'POST', 'PUT')`
- HTTPS: `->secure()`
- regex for matching a path parameter: `->pattern($parameter, $regex)`
    ```php
    '/user/{id}' => route('controller')->pattern('id', '\d+')
    ```
- optional path parameters: `->optional($parameter, $defautValue)`
    ```php
    '/feed{format}' => route('controller')->optional('format', '.json')
    ```

### Controllers

Controllers can be any PHP callable or **extended callable** ([read the documentation](../extended-callables.md)).

#### Return value

A controller can return a response (like any middleware) or **a string** (which will be written to the response):

```php
function ($request, $response, $next) {
    return 'Hello world';
}
```

#### Parameters

The parameters passed to the controller are matched based on what parameters the controller expect. The following parameters can be used (from highest priority to lowest):

- middleware parameters: `$request`, `$response`, `$next`
- route path parameters
- dependency injection using type-hints

The list is from the highest priority to the lowest in case of collisions (e.g. if one of your route parameter is named `request` then the HTTP request will be injected, not the route parameter).

For example you can use the classic middleware signature:

```php
'/{id}' => function ($request, $response, $next) {
    $id = $request->getAttribute('id');
    ...
}
```

Or expect the route parameter directly:

```php
'/{id}' => function ($id) {
    ...
}
```

You can type-hint services to inject them:

```php
'/' => function (Authentication $auth) {
    return 'Welcome ' . $auth->getCurrentUserName();
}
```

Or even mix everything (the order doesn't matter):

```php
'/{id}' => function ($id, $response, Authentication $auth) {
    ...
}
```

Note that injecting services using type-hints will only work with container entries whose name is a class (or interface) name. You'll also find that dependency injection through parameters isn't very useful when your controller is a class (because in that case you can do classic dependency injection in the constructor).

### Run

The router can be invoked directly:

```php
$router($request, $response, $next);
```

Like any other middleware, it takes a PSR-7 request, response and the next callable to invoke if no route matched. In most application, the `next` callable is usually returning a 404 response.

The router can also be added to a middleware-based application (like any other middleware) - this is how it is used in the Stratify framework.

## The prefix router

The `Stratify\Router\PrefixRouter` is a very small and lightweight router that does only one thing: route to sub-middlewares based on a URL prefix.

```php
$router = new PrefixRouter([
    '/api/'   => function ($request, $response, $next) {
        return ...;
    },
    '/admin/' => function ($request, $response, $next) {
        return ...;
    },
]);
```

In this example, all URLs starting with `/api/` will be routed to the first middleware, and all starting with `/admin/` will be routed to the second one.

It is useful if you want to apply some middlewares only for specific parts of an application, for example:

- protect an `/admin/` section with an authentication middleware
- cache the front-end part of an app with a cache middleware
- use a "REST router" for the `/api/` part of the application, and use a classic router for the rest of the website

Here is a more advanced example where we protect only the `/admin/` section with authentication:

```php
$router = new PrefixRouter([

    '/api/'   => ...,

    '/admin/' => new MiddlewarePipe([
        function ($request, $response, $next) {
            // check authentication
        },
        new Router([
            '/admin/login' => [LoginController::class, 'login'],
        ]),
    ]),

]);
```

However as you start to dive in such use cases, you are encouraged to look at the full Stratify framework which offers features dedicated to compose applications in that way.

Note: this router is an alternative to other framework's features like registering a middleware for a URL prefix, or route middlewares. Instead of having a do-it-all core that covers specific use cases (route middleware, path middleware, …), Stratify exposes one simple pattern (middleware) and lets you create complex behaviors by assembling very simple bricks (like the "Unix philosophy").
