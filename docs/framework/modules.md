---
layout: documentation
---

# Modules

A Stratify module boils down to:

- a lower-case name (e.g. `my-module`)
- a [Puli package](http://docs.puli.io/en/latest/glossary.html#glossary-package) whose Puli root path is the module name (e.g. `/my-module`)
- a [PHP-DI](http://php-di.org/) configuration file inside that package (located at `/my-module/config/config.php`)

A module can also contain more things like PHP classes (e.g. middlewares), resources (views, JS, CSS, …).

The Puli package allows to expose resources (views, JS, CSS, …). It can be seen as PSR-4/autoloading for resources. That will be used for example to find and load the module's configuration file.

With the configuration, a module is able to expose pre-configured objects (services, middlewares, …) and offer extension points on these objects.

## Using a module

To use a module, you need to install it with Composer and then register it in the application:

```php
$modules = [
    'module1',
    'module2',
    'module3',
    'app',
];

$app = new Application($http, $modules);
```

That will automatically register the Puli packages as well as PHP-DI's config files.

Note that each module config is registered in the order of the list. If some modules override others, you need to register them *after* the module they override. For example, it is highly recommended to register the `app` module last to be able to override everything in your local configuration.

## The `app` module

While modules can be reusable and isolated Composer packages, they can also be full applications.

As a matter of fact, you should write an application-style project *as a Stratify module*. Just like [Symfony's `AppBundle`](http://symfony.com/doc/current/best_practices/creating-the-project.html#application-bundles), you are encouraged to use the generic `app` name for that application module.

## Writing a module

### File layout

```
src/
    ...
res/
    config/
        config.php
    ...
composer.json
puli.json
```

The `composer.json` file contains the package information (name, license, …) as well as PSR-4 configuration to autoload PHP classes stored in `src/`.

The `puli.json` file contains configuration for mapping virtual Puli paths (`/my-module/…`) to filesystem paths (`res/…`). To create it, use the `puli map` command:

```bash
puli map /my-module res/
```

### Configuration

The module's configuration is stored in `res/config/config.php`. The syntax you can use is documented in [PHP-DI's documentation](http://php-di.org/doc/php-definitions.html).

#### Example

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

Note that this example shouldn't be actually used: Stratify provides a Twig module already.

### Resources

Resources are stored in the `res/` directory. Resources can be anything that isn't a PHP class: configuration, assets, translations, views, … You are encouraged to follow [Puli's recommended file layout](http://docs.puli.io/en/latest/repository/directory-layout.html):

```bash
res/
    config/
    public/
        css/
        js/
        images/
    trans/ # translation files
    views/ # Twig/Blade/… templates
```

All module resources can be accessed using Puli and its virtual "Puli paths", for example: `/module-name/public/js/main.js`. Read more about it in [Puli's documentation](http://docs.puli.io/en/latest/repository/introduction.html#how-it-works).
