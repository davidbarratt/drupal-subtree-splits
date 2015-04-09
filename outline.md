# Using Subtree Splits to spread Drupal into everything

## Outline

### Dependency Management
#### [Composer](https://getcomposer.org)
##### Basic `composer.json` file
```json
  {
  "require": {
    "guzzlehttp/guzzle": "~5.0"
  }
}
```
*Pro tip:* you can just execute `composer require guzzlehttp/guzzle ~5.0`
##### Executing `composer install` will download the `guzzle` into `./vendor`
##### Guzzle's `composer.json` looks like this:
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
Because Guzzle depends on `guzzlehttp/ringphp` it will also download that.
    * Include the Composer autoloader in your PHP file:
      ```php
        require 'vendor/autoload.php';
      ```
      then you can just use it:
      ```php
        $client = new GuzzleHttp\Client();
        $response = $client->get('http://guzzlephp.org');
      ```
  * Sub Projects (Components)
    * Symfony
    * Laravel
  * Drupal

* Other Projects use/maintain subtree splits
  * Why?
  * How?
* Drupal.org should maintain splits of Drupal
  * Why?
  * How?
* Others can use these splits in their own projects
* Benefits the community can get from offering these splits
