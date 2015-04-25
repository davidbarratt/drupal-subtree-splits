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

##### Locking Dependencies
Whenever you run `composer install` for the first time, composer generates a
`composer.lock` file. This file specifies the precise commit hash of dependency
you just installed.

From [Composer](http://getcomposer.org/doc/01-basic-usage.md#composer-lock-the-lock-file):
"This file locks the dependencies of your project to a known state"

Next time someone runs `composer install`, Composer essentially ignores what is
in `composer.json` and looks at `composer.lock`. The only time `composer.lock`
is updated is when running `composer update`.

Composer, however, will warn you when the versions in your lock file are out of
sync with versions in your json file.

For instance, if you change `guzzlehttp/guzzle` from `~5.0` to `~4.0`, what is
is in the lock file is technically no longer valid and Composer will throw this
warning:
```
Warning: The lock file is not up to date with the latest changes in
composer.json. You may be getting outdated dependencies. Run update to update
them.
```
but it still follows the lock file.

Because of the lock file, we can ensure that every developer is running the
exact same versions of the dependencies as everyone else is.

There are only two files that should be commited to version control:
```
composer.json
```
and
```
composer.lock
```
everything else is unnecessary.

*NOTE:* The `composer.lock` file only applies to the current working directory,
Composer ignores the `composer.lock` in all of your dependencies.

Because of this, many committing the `composer.lock` file is optional and many
libraries (like Guzzle) do not.

##### Drupal Core as a Consumer

###### Proposal #1:
[#1475510 Remove external dependencies from the core repo and let Composer
manage the dependencies instead](https://www.drupal.org/node/1475510)
We should remove the `core/vendor` directory from the repository. This allows us
to manage dependencies without *huge* patches. The only thing that needs to
happen, is that the test bot & packaging script needs to execute
`cd core; composer install;` on every run so the dependencies are available
during testing and are included in any downloadable packages. This removes
14MB (!) from our repository. Composer caches dependencies that you download, so
there shouldn't be any increase in network traffic, it will just copy the
dependency from it's cache into your project.

This also prevents Composer users from having to have two copies of the
dependencies (more on this later).

###### Proposal #2:
[#2444615 Move phpunit and mink to require-dev](https://www.drupal.org/node/2444615)
There are some dependencies that are not needed unless you are running tests.
PHPUnit, Mink, etc. These dependencies could be moved to Comoposer's
[require-dev](https://getcomposer.org/doc/04-schema.md#require-dev).

Drupal 8's `core/composer.json` file would then look like this:
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
    "zendframework/zend-feed": "2.4.*",
    "mikey179/vfsStream": "1.*",
    "stack/builder": "1.0.*",
    "egulias/email-validator": "1.2.*"
  },
  "require-dev": {
    "phpunit/phpunit": "4.4.*",
    "behat/mink": "~1.6",
    "behat/mink-goutte-driver": "~1.1",
    "fabpot/goutte": "^2.0.3",
    "masterminds/html5": "~2.1"
  }
}
```
Why would we do this? Assuming Proposal #1 is accepted, you could run this
command for the test-bot:
```
cd core; composer install;
```
and this command for the packaging script:
```
cd core; composer install --no-dev
```
The later command excludes the dependencies in `require-dev`. This makes our
package smaller and more secure since most Drupal users do not need these
dependencies, there is little reason for them being in downloaded packages.

###### Proposal #3:
**Require Drush as a dev dependency**
You can now install drush on a [per-project basis](http://docs.drush.org/en/master/install/).
This means drush can be managed the same way PHPUnit, Mink, etc. is managed.
We should add drush as a dev dependency, so we can ensure that every Drupal
contributor is running the same version of drush. The reduces the number of
variables when attempting to reproduce bugs.

Assuming Propsal #1 & #2 is accepted, we just need to add this to
`core/composer.json`:
```json
"require-dev": {
  "drush/drush": "~7.0"
}
```

Because of Propsal #1 & #2, we can install the same version of drush for every
contributor, but not for every user of Drupal.

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
