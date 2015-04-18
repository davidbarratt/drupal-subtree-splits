# Using Subtree Splits to spread Drupal into everything

## Outline

### Dependency Management
[npm](https://www.npmjs.com/)
[bower](http://bower.io/)
[bundler](http://bundler.io/)
#### [Composer](https://getcomposer.org)
Composer is a tool for dependency management in PHP. It allows you to declare
the dependent libraries your project needs and it will install them
in your project for you.

### Consumers and Providers

#### Consumers
Projects that consume libraries that via Composer.

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

Drupal 8's `core/composer.json` file looks like this:
```json
{
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

We can now consume all of these dependencies and all of these libraries'
dependencies.

(something about composer.lock here)

[#1475510 Remove external dependencies from the core repo and let Composer
manage the dependencies instead](https://www.drupal.org/node/1475510)
We should remove the `core/vendor` directory from the repository. This allows us
to manage dependencies without *huge* patches. The only thing that needs to
happen, is that the test bot & packaging script needs to execute
`cd core; composer install;` on every run so the dependencies are available
during testing and are included in any downloadable packages. 


<!--
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
-->
