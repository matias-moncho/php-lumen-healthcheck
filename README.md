# Laravel Health Check  

[![Build Status](https://github.com/Saritasa/php-laravel-healthcheck/workflows/build/badge.svg)](https://github.com/Saritasa/php-laravel-healthcheck/actions)
[![CodeCov](https://codecov.io/gh/Saritasa/php-laravel-healthcheck/branch/master/graph/badge.svg)](https://codecov.io/gh/Saritasa/php-laravel-healthcheck)
[![Release](https://img.shields.io/github/release/saritasa/php-laravel-healthcheck.svg)](https://github.com/Saritasa/php-laravel-healthcheck/releases)
[![PHPv](https://img.shields.io/packagist/php-v/saritasa/laravel-healthcheck.svg)](http://www.php.net)
[![Downloads](https://img.shields.io/packagist/dt/saritasa/laravel-healthcheck.svg)](https://packagist.org/packages/saritasa/laravel-healthcheck)

Package for Laravel-based project self-diagnostics. 
Implements basic checks (ex. if application can connect to DB server)
and allows extensibility (ex. implement custom checks) 
  
## Laravel 5.5+
  
Install the ```saritasa/laravel-healthcheck``` package:  
  
```bash  
$ composer require saritasa/laravel-healthcheck  
```  

## Configuration
- Publish configuration file:

```bash
php artisan vendor:publish --provider="Saritasa\LaravelHealthCheck\HealthCheckServiceProvider"
```

Configure the necessary checks in file `config/health_check.php`

```php
    'checkers' => [
        'database' => \Saritasa\LaravelHealthCheck\Checkers\DatabaseHealthChecker::class,
        'redis' => \Saritasa\LaravelHealthCheck\Checkers\RedisHealthChecker::class,
        's3' => \Saritasa\LaravelHealthCheck\Checkers\S3HealthChecker::class,
    ],
```  

You can add more custom checks - just add a class implementing 
`\Saritasa\LaravelHealthCheck\Contracts\ServiceHealthChecker` interface with single method `check()` 
that must return instance of `\Saritasa\LaravelHealthCheck\Contracts\CheckResult`.

## Laravel Lumen 8.0+

Install the ```saritasa/laravel-healthcheck``` package:  
  
```bash  
$ composer require saritasa/laravel-healthcheck  
```  

Create a new file ```config\health_check.php```:
```php
<?php

use Saritasa\LaravelHealthCheck\Checkers\DatabaseHealthChecker;
use Saritasa\LaravelHealthCheck\Checkers\RedisHealthChecker;
use Saritasa\LaravelHealthCheck\Checkers\S3HealthChecker;

return [
    'checkers' => [
        'database' => DatabaseHealthChecker::class,
        'redis' => RedisHealthChecker::class,
        's3' => S3HealthChecker::class,
    ],
];
```

Add the service provider in file ```bootstrap\app.php```:

```php
$app->configure('health_check');

$app->instance('path.config', app()->basePath() . DIRECTORY_SEPARATOR . 'config');
$app->register(Saritasa\LaravelHealthCheck\HealthCheckServiceProvider::class);
``` 

# Usage
Package exposes endpoints to run all checks or run each check by name:
## GET /health
Runs all known checks and returns HTTP code = 200, if all checks succeeded, 500 otherwise.  
Response contains JSON with pares of check name and true/false indicating if checker completed successfully or not.

## GET /health/{checker}
Where **{checker}** is a key from `config/health_check.php`, ex. `GET /health-check/database`.  
Returns HTTP code = 200, if checker reports success, 500 otherwise.  
Returns payload, returned by checker. If check result is not successful, adds error message.

## GET /liveness, GET /readness
Do nothing, just check availability of PHP application

## Available checkers
#### Saritasa\LaravelHealthCheck\Checkers\DatabaseHealthChecker  
Checks, if default connection to DB, configured in Laravel is available - tries to establish connection to server.

#### Saritasa\LaravelHealthCheck\Checkers\RedisHealthChecker  
Checks, if redis connection is available - tries to establish connection to server.

#### Saritasa\LaravelHealthCheck\Checkers\S3HealthChecker  
Checks, if application can read from default S3 bucket - tries to get enumerate entries in S3 bucket.

#### Saritasa\LaravelHealthCheck\Checkers\NullChecker
Does nothing. Use if you need to check HTTP server availability only.

## Contributing  
  
1. Create fork, checkout it  
2. Develop locally as usual. **Code must follow [PSR-1](http://www.php-fig.org/psr/psr-1/), [PSR-2](http://www.php-fig.org/psr/psr-2/)** -  
    run [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) to ensure, that code follows style guides  
3. **Cover added functionality with unit tests** and run [PHPUnit](https://phpunit.de/) to make sure, that all tests pass  
4. Update [README.md](README.md) to describe new or changed functionality  
5. Add changes description to [CHANGES.md](CHANGES.md) file. Use [Semantic Versioning](https://semver.org/) convention to determine next version number.  
6. When ready, create pull request  
