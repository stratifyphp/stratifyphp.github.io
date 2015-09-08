---
layout: documentation
---

# Extended callables

## Classic PHP callables

PHP has the concept of [**callables**](http://php.net/manual/en/language.types.callable.php), i.e. something that can be invoked.

Here is a (non-exhaustive) list of valid PHP callables:

```php
// Closure
$callable = function() { ... }

// Function
function my_function() { ... }
$callable = 'my_function';

// Static method
class MyClass
{
    public static function foo() { ... }
}
$callable = ['MyClass', 'foo'];

// Method
class MyClass
{
    public function foo() { ... }
}
$callable = [new MyClass(), 'foo'];

// Invokable class
class MyClass
{
    public function __invoke() { ... }
}
$callable = new MyClass();
```

PHP callables can be invoked simply:

```php
$callable();
```

## Extended callables

Using PHP callables in a framework (e.g. for controllers) is great because it lets the user free to choose which form to use. For example it allows to start very easily with closures and then move to classes when the application grows.

However **this is limited**: using an invokable class or object method as a controller means you have to create the object (`[new MyClass(), 'foo']` or `new MyCallableClass()`):

- performance-wise this means a lot of objects are created during the application's bootstrap and maybe not used
- it's harder to use dependency injection in controllers

To solve these issues, Stratify introduces the concept of **extended callables**: these callables are not actual PHP callables, but they reuse the same principles.

For those familiar with other frameworks, this is a friendlier alternative to custom notations to define controllers as services (e.g. `Acme\App\UserController:list` in [Symfony](http://symfony.com/doc/current/cookbook/controller/service.html#referring-to-the-service), [Slim](https://github.com/slimphp/Slim/blob/3.x/Slim/CallableResolver.php#L50-L51), [Silex](http://silex.sensiolabs.org/doc/providers/service_controller.html#usage), `Acme\App\UserController@list` in [Laravel](http://laravel.com/docs/5.1/controllers#basic-controllers), `Acme\App\UserController::list` in ZF2, …).

### Object method call

The PHP callable for a method call is `[new MyClass(), 'foo']`. The extended callable for this is:

```php
class MyClass
{
    public function foo() { ... }
}

$callable = [MyClass::class, 'foo'];
```

The first item of the array can be *any container entry*. That means it can be a class name or anything else (e.g. `['db', 'getConnection']`).

When the callable is invoked, the object **will be created by the container** (so you can use any dependency injection feature). That is equivalent to the following code:

```php
$object = $container->get(MyClass::class);
// Now we have a valid PHP callable
$callable = [$object, 'foo'];
```

You might notice that this notation is similar to a *static* method call `['Foo', 'bar']`. It is indeed the same. However there is no collision: if the method is static, then the method will be invoked statically without creating an instance. If the method is not static, then it's an *extended callable*.

### Invokable class

The PHP callable for an invokable class (implementing `__invoke()`) is `new MyClass()`.

The extended callable for this is simply the class name (or *any container entry*):

```php
class MyClass
{
    public function __invoke() { ... }
}

$callable = MyClass::class;
```

When the callable is invoked, the object **will be created by the container** (so you can use any dependency injection feature), and then invoked.
