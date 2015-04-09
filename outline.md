# Using Subtree Splits to spread Drupal into everything

## Outline

### Dependency Management
#### [Composer](https://getcomposer.org)
Basic `composer.json` file
```json
  {
  "require": {
    "guzzlehttp/guzzle": "~5.0"
  }
}
```
*Pro tip:* you can just execute `composer require guzzlehttp/guzzle ~5.0`

Executing `composer install` will download the `guzzle` into `./vendor`

[Guzzle's](http://guzzlephp.org) `composer.json` looks like this:
```json
{
  "name": "guzzlehttp/guzzle",
  "type": "library",
  "license": "MIT",
  "require": {
      "php": ">=5.4.0",
      "guzzlehttp/ringphp": "~1.0"
  },
}
```

Because [Guzzle](http://guzzlephp.org) depends on `guzzlehttp/ringphp` it will
also download that.

Include the Composer autoloader in your PHP file:
```php
require 'vendor/autoload.php';
```

then you can just use it:
```php
$client = new GuzzleHttp\Client();
$response = $client->get('http://guzzlephp.org');
```

This has completely changed the game. No longer are we reinventing the wheel.
[Composer](https://getcomposer.org) and the
[PSR-0](http://www.php-fig.org/psr/psr-0/) and
[PSR-4](http://www.php-fig.org/psr/psr-4/) autoloading standards have allowed
Drupal to "get off the island". We now use a large number of third-party
libraries that would have been inaccessible before.

Drupal 8's `composer.json` file looks like this:
```json
{
  "name": "drupal/core",
  "type": "drupal-core",
  "license": "GPL-2.0+",
  "require": {
    "php": ">=5.4.5",
    "sdboyer/gliph": "0.1.*",
    "symfony/class-loader": "2.6.*",
    "symfony/css-selector": "2.6.*",
    "symfony/dependency-injection": "2.6.*",
    "symfony/event-dispatcher": "2.6.*",
    "symfony/http-foundation": "2.6.*",
    "symfony/http-kernel": "2.6.*",
    "symfony/routing": "2.6.*",
    "symfony/serializer": "2.6.*",
    "symfony/validator": "2.6.*",
    "symfony/process": "2.6.*",
    "symfony/yaml": "2.6.*",
    "twig/twig": "1.16.*",
    "doctrine/common": "~2.4.2",
    "doctrine/annotations": "1.2.*",
    "guzzlehttp/guzzle": "~5.0",
    "symfony-cmf/routing": "1.3.*",
    "easyrdf/easyrdf": "0.9.*",
    "phpunit/phpunit": "4.4.*",
    "zendframework/zend-feed": "2.4.*",
    "mikey179/vfsStream": "1.*",
    "stack/builder": "1.0.*",
    "egulias/email-validator": "1.2.*",
    "behat/mink": "~1.6",
    "behat/mink-goutte-driver": "~1.1",
    "fabpot/goutte": "^2.0.3",
    "masterminds/html5": "~2.1"
  }
}
```

How do we allow users to use Drupal in the same way we use all these libraries?

# Frameworks
Symfony
Laravel

# Drupal
Root Project
Core
Components
Modules
Contrib
Packagist

* Other Projects use/maintain subtree splits
  * Why?
  * How?
* Drupal.org should maintain splits of Drupal
  * Why?
  * How?
* Others can use these splits in their own projects
* Benefits the community can get from offering these splits
