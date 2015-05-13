# Using Subtree Splits to spread Drupal into everything

## Outline

### Clarification
Drupal: Everything except the `/core` folder.
Drupal core: Everything in the `/core` folder.

### Dependency Management
[npm](https://www.npmjs.com/)
[bower](http://bower.io/)
[bundler](http://bundler.io/)

Drupal actually has had a dependency management tool for a long time. It's one
of Drupal's best features. Module developers have the ability to declare
dependencies on other modules. This single feature, I believe, is responsible
for the huge number of "API" modules (Chaos Tools, Entity API, Libraries, etc.).
There is no other major CMS (that I am aware of) that has this feature.

What if we want to go beyond modules and declare dependencies for almost
anything?

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
Drupal core to "get off the island". We now use a large number of third-party
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

Also, allows us to run tests against different version of dependencies.
(i.e. Symfony 2.6 and 2.7, or whatever Drupal wants to be compiable with).

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
We should add drush as a dev dependency, so we can ensure that every Drupal core
contributor is running the same version of drush. The reduces the number of
variables when attempting to reproduce bugs.

Assuming Propsal #1 & #2 is accepted, we just need to add this to
`core/composer.json`:
```json
{
  "require-dev": {
    "drush/drush": "~7.0"
  }
}
```

Because of Propsal #1 & #2, we can install the same version of drush for every
contributor, but not for every user of Drupal.

#### Provider
A Provider is a package (typicaly a library) that can be consumed by another
package. What makes a package a provider? A name.
```json
{
  "name": "guzzlehttp/guzzle",
}
```
(Technially this is all we need, but Packagist requires a description as well).
```json
{
  "name": "guzzlehttp/guzzle",
  "description": "Guzzle is a PHP HTTP client library and framework for building RESTful web service clients"
}
```

It's a good practice to give a name and a description to every package, even if
it's never going to be consumed.

Now that we have a name and a description, we can submit the package to
[Packagist](https://packagist.org/).

##### Drupal Core as a Provider
Why would anyone want to consume Drupal core? It's not a library!

Drupal core does have it's own dependencies, why not let Composer manage these
for us? If we don't let Composer manage the dependencies, we could end up with
dependencies that our out of sync (i.e. wrong version of symfony components,
etc.)

From `core/composer.json`
```json
{
  "name": "drupal/core",
  "description": "Drupal is an open source content management platform powering millions of websites and applications.",
  "type": "drupal-core"
}
```
The type tells Composer that this is something other than a library.

Now any Drupal project can load Drupal core by adding:
```json
{
  "require": {
    "drupal/core": "~8.0"
  }
}

```
*Pro tip:* you can just execute `composer require drupal/drupal ~8.0`

This isn't quite right though, if we run `composer install` drupal/core will be
installed in `vendor/drupal/core`. Drupal does not work in this directory since
it needs to be in `/core`. To get around this, we'll also require
`composer/installers`.

[Composer Installers](http://composer.github.io/installers/) is a Composer
plugin that installs packages of a specific type in a specific directory.

In our case `drupal-core` will be installed in `/core`.

Now our `composer.json` will look something like this:
```json
{
  "require": {
    "composer/installers": "^1.0.20",
    "drupal/core": "~8.0"
  }
}
```

Normally you would separate the project (Drupal) from the framework
(Drupal Core). This is what they've done with
[Symfony](https://github.com/symfony/symfony) (the framework) and [Symfony Standard](https://github.com/symfony/symfony-standard)
(the project). As well as with [Laravel](https://github.com/laravel/laravel)
(the project) and [Larval Framework](https://github.com/laravel/framework)
(the framework). In both of these instances, the project is just a starting
point.

The "project" is actually denouted by a special composer type:
```json
{
  "type": "project"
}
```

Take a look at the `./composer.json` and `./core/composer.json` and you'll see
what I'm talking about.

###### Proposal #4:
[#2385387 Permanently split Drupal and Drupal core into seperate repositories](https://www.drupal.org/node/2385387).
Would bring us in line with other projects and frameworks, but would require a
re-roll of every patch out there. This isn't going to happen until Drupal 9 at
the earliest. But it would allow for more than one project/distrobution to be
be created with the same Drupal core.

Since we can't do that, we'll need to fake it.

###### Proposal #5:
[#2385387 Create (and maintain) a subtree split of Drupal cores](https://www.drupal.org/node/2385387).
This is a stop-gap solution. Instead of splitting the repositories, we can
create and maintain a subtree split of Drupal core. This allows developers
to consume Drupal core while at the same time maintaining a single repository
for contributors.

Right now, [tstoeckler](https://www.drupal.org/u/tstoeckler) is maintaining the
split for the community. Everyone who installs Drupal core with composer will be
using his split.

The read-only repository should live at: https://drupal.org/project/core

###### Proposal #6:
**Split each Component into a read only repository.**
This is where the magic happens. When the Drupal 8 components
(`core/lib/Drupal/Components`) are split into read-only repositories, then
other PHP projects can depend on our components. In the same way we depend on
[Symfony Components](http://symfony.com/components), Symfony could depend on
Drupal components. Wouldn't it be impressive if other CMSs depended on Drupal
components?

This is how Drupal can be spread into everything. Just as we have split modules
into smaller "API" modules, Drupal core can be split into smaller read-only
"Components". These components can be consumed by any PHP project. The more
high-quality components that are available, the more likely they will be used
in libraries. This drives interest in Drupal as a whole. Just as the Symfony
components being used in Drupal has driven intrest in Symfony as a whole.

##### Contrib (and Modules)

###### Consumers

What if a contrib module wants to depend on 3rd-party library?

Normally, they'd just require it in their `composer.json` file:
```json
{
  "require": {
    "guzzlehttp/oauth-subscriber": "0.2.*"
  }
}
```

This is great, but how do my users install my module?

Two choices:

[Composer Manager](https://www.drupal.org/project/composer_manager)
This module is a little "magical". It takes all of the `composer.json` files
from all of your files and combines them into a single file. This technically
works, but it can get really complicated if, later, you'd like to modify
`composer.json` yourself.

Load the whole module with Composer. Since Composer gets the dependencies
recursively, Composer will download your module(s) as well as your module(s)
dependencies.


####### Drupal Packagist
Drupal module's version numbers (7.x-1.0) are not compatible with Composer's
X.Y.Z (or W.X.Y.Z) format. To get around this,
[Drupal Packagist](https://packagist.drupal-composer.org/) was created which
translates 7.x-1.0 to 7.1.0. It also lists every module available on Drupal.org

###### Proposal #6:
**Provide a Release Webhook & Better APIs**
Some pieces of data are scraped from Drupal.org rather than using an API.
Also, Packagist needs a webhook so they can be notified when a module has a new
release, rather than checking every single module on a periodic basis.

###### Proposal #7:
[#1612910 Switch to Semantic Versioning for Drupal contrib extensions (modules, themes, etc)](https://www.drupal.org/node/1612910)
Rather than translating the version numbers in Drupal Packagist, we could
just permanently move all contrib modules to follow a X.Y.Z standard (like Core).
Then we could potentially use the standard Packagist.org, however, that will
require us publish every module there.

###### Proposal #8:
Move packagist to packagist.drupal.org. Pretty self-explanatory, make Drupal Packagist an
official part of Drupal.org. This way we can include it in `./composer.json` and
reference Drupal Packagist in official documentation.

###### Providers

What if a module would like to provider read-only subtree splits of their cmodules
and libraries?

###### Proposal #8:
**Allow contrib developers to define subfolders (modules or libraries) that should have
subtree splits**
Ideally, these splits should be maintined by a Drupal.org system for the maintainer. This
way there isn't any scripts or anything a maintainer has to run, they can simply define
the splits on the module and those will be created. These could be sub-module or libraries.

This will help drive traffic to Drupal.org by people looking for libraries for their
PHP project.

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
