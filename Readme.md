# Laravel-Roadrunner
Simple Laravel-Roadrunner bridge 

## Install

```shell
$ composer require hunternnm/laravel-roadrunner
```


### Example `psr-worker.php`
```php
<?php

use Hunternnm\LaravelRoadrunner\RoadrunnerLaravelBridge;
use Illuminate\Foundation\Application;
use Illuminate\Http\Request;
use Spiral\Goridge;
use Spiral\RoadRunner;
use Symfony\Bridge\PsrHttpMessage\Factory\HttpFoundationFactory;
use Symfony\Bridge\PsrHttpMessage\Factory\PsrHttpFactory;
use Zend\Diactoros\ResponseFactory;
use Zend\Diactoros\ServerRequestFactory;
use Zend\Diactoros\StreamFactory;
use Zend\Diactoros\UploadedFileFactory;

ini_set('display_errors', 'stderr');
require 'vendor/autoload.php';

$worker = new RoadRunner\Worker(new Goridge\StreamRelay(STDIN, STDOUT));
$psr7 = new RoadRunner\PSR7Client($worker);
$httpFoundationFactory = new HttpFoundationFactory();

/** @var Application $app */
$app = require __DIR__.'/bootstrap/app.php';

$rr = new RoadrunnerLaravelBridge($app, __DIR__);

$psr7factory = new PsrHttpFactory(
    new ServerRequestFactory(),
    new StreamFactory(),
    new UploadedFileFactory(),
    new ResponseFactory()
);

while ($req = $psr7->acceptRequest()) {
    try {
        $symfonyRequest = $httpFoundationFactory->createRequest($req);
        $request = Request::createFromBase($symfonyRequest);

        $response = $rr->request($request);

        $psr7response = $psr7factory->createResponse($response);
        $psr7->respond($psr7response);
    } catch (Throwable $e) {
        $psr7->getWorker()->error((string) $e);
    }
}

```