---
layout: documentation
---

# Templating

TODO: Twig introduction

## Installation

```
composer require stratify/twig-module
```

```php
$modules = [
    // ...
    'twig',
];
```

## Render a template

Inject `Twig_Environment` and call the `render()` method by passing a Puli path:

```php
return $twig->render('/app/views/home.twig');
```

## Writing a template

TODO: link to Twig documentation

## Assets

### Publish assets with Puli

Store assets in `res/public/`. Puli can then publish them for you (using symlinks):

```
puli server --add localhost web
puli publish /app/public localhost
puli publish --install
```

### Link assets in Twig views

The `resource_url()` Twig function:

```html
<link rel="stylesheet" href="{{ resource_url('/app/public/css/bootstrap.min.css') }}">
```
