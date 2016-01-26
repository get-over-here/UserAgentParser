# UserAgentParser

[![Build Status](https://travis-ci.org/ThaDafinser/UserAgentParser.svg)](https://travis-ci.org/ThaDafinser/UserAgentParser)
[![Code Coverage](https://scrutinizer-ci.com/g/ThaDafinser/UserAgentParser/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/ThaDafinser/UserAgentParser/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/ThaDafinser/UserAgentParser/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/ThaDafinser/UserAgentParser/?branch=master)
[![PHP 7 ready](http://php7ready.timesplinter.ch/ThaDafinser/UserAgentParser/badge.svg)](https://travis-ci.org/ThaDafinser/UserAgentParser)

[![Latest Stable Version](https://poser.pugx.org/thadafinser/user-agent-parser/v/stable)](https://packagist.org/packages/thadafinser/user-agent-parser)
[![Latest Unstable Version](https://poser.pugx.org/thadafinser/user-agent-parser/v/unstable)](https://packagist.org/packages/thadafinser/user-agent-parser) 
[![License](https://poser.pugx.org/thadafinser/user-agent-parser/license)](https://packagist.org/packages/thadafinser/user-agent-parser)
[![Total Downloads](https://poser.pugx.org/thadafinser/user-agent-parser/downloads)](https://packagist.org/packages/thadafinser/user-agent-parser) 

`User agent` parsing is, was and will always be a painful thing.

The target of this package is to make it less painful, by providing an abstract layer for many user agent parsers.

So you can
- use multiple providers at the same time with the `Chain` provider
- use local and/or HTTP API providers at the same time
- switch between different parsers, without changing your code
- compare the result of the different parsers
- get always the same result model, regardless of which parser you use currently


## Try it out

[LIVE test](http://useragent.mkf.solutions/)

[Comparison matrix](http://thadafinser.github.io/UserAgentParserComparison/)


## Installation

Using composer is currently the only supported way to install this package.

```
composer require thadafinser/user-agent-parser
```

`Note:` to use local providers you need to install additional packages, which are listed inside the composer `suggests section`


## Getting started

After you have installed the package, you can use currently only `UserAgentStringCom` out of the box.
For all other providers, you need to register an API key or install an additional package (listed in the section `suggest` of `composer.json`)


```php
use UserAgentParser\Exception\NoResultFoundException;
use UserAgentParser\Provider\Http\UserAgentStringCom;
use GuzzleHttp\Client;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Handler\CurlHandler;

$client = new Client([
    'handler' => HandlerStack::create(new CurlHandler()),
]);

$provider = new UserAgentStringCom($client);

try {
    /* @var $result \UserAgentParser\Model\UserAgent */
    $result = $provider->parse('Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.73 Safari/537.36');
} catch (NoResultFoundException $ex){
    // nothing found
}

if($result->getBot()->getIsBot() === true) {
  // if one part has no result, it's always set not null
  $result->getBot()->getName();
  $result->getBot()->getType();
} else {
  // if one part has no result, it's always set not null
  $result->getBrowser()->getName();
  $result->getBrowser()->getVersion()->getComplete();

  $result->getRenderingEngine()->getName();
  $result->getRenderingEngine()->getVersion()->getComplete();

  $result->getOperatingSystem()->getName();
  $result->getOperatingSystem()->getVersion()->getComplete();

  $result->getDevice()->getModel();
  $result->getDevice()->getBrand();
  $result->getDevice()->getType();
  $result->getDevice()->getIsMobile();
  $result->getDevice()->getIsTouch();
}
```

## Providers

UserAgentParser comes with local and http providers


### Local providers

Local providers are (most time) faster then HTTP providers and dont require a working internet connection.
But you need to update them yourself from time to time, to make sure you detect the latest UAs

- BrowscapPhp
- DonatjUAParser
- PiwikDeviceDetector
- SinergiBrowserDetector
- UAParser
- WhichBrowser
- Woothee
- Wurfl


### HTTP providers (API)

HTTP providers are simple to use, since you need only an API key to get started.
But they require (always) a working internet connection.

- Http\DeviceAtlasCom
- Http\NeutrinoApiCom
- Http\UdgerCom
- Http\UserAgentApiCom
- Http\UserAgentStringCom (no API key required)
- Http\WhatIsMyBrowserCom


### Comparison matrix

Here is a comparison matrix, with many analyzed UserAgent strings, to help you device which provider fits your needs.
Every provider has it's strengh and weakness, so it will depend on your need, which one you should use.

[Go to the matrix](https://github.com/ThaDafinser/UserAgentParserMatrix)

### Overview

### Single provider

```php
require 'vendor/autoload.php';

use UserAgentParser\Provider;

$userAgent = 'Mozilla/5.0 (iPod; U; CPU iPhone OS 4_3_5 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8J2 Safari/6533.18.5';

$provider = new Provider\PiwikDeviceDetector();

/* @var $result \UserAgentParser\Model\UserAgent */
$result = $provider->parse($userAgent);
// optional add all headers, to improve the result further
// $result = $provider->parse($userAgent, getallheaders());

$result->getBrowser()->getName(); // Mobile Safari
$result->getOperatingSystem()->getName(); // iOS
$result->getDevice()->getBrand(); // iPod Touch
$result->getDevice()->getBrand(); // Apple
$result->getDevice()->getType(); // portable media player

$resultArray = $result->toArray();
```

### Chain provider

This is very useful to improve your results.
The chain provider starts with the first provider and checks if there is a result, if not it takes the next one and so on.
If none of them have a result, it will throw a NoResultException like a single provider.

```php
require 'vendor/autoload.php';

use UserAgentParser\Provider;

$userAgent = 'Mozilla/5.0 (iPod; U; CPU iPhone OS 4_3_5 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8J2 Safari/6533.18.5';

$chain = new Provider\Chain([
    new Provider\PiwikDeviceDetector(),
    new Provider\WhichBrowser(),
    new Provider\UAParser(),
    new Provider\Woothee(),
    new Provider\DonatjUAParser()
]);

/* @var $result \UserAgentParser\Model\UserAgent */
$result = $chain->parse($userAgent);
// optional add all headers, to improve the result further (used currently only by WhichBrowser)
//$result = $chain->parse($userAgent, getallheaders());

$result->getBrowser()->getName(); // Mobile Safari

$result->getOperatingSystem()->getName(); // iOS

$result->getDevice()->getBrand(); // iPod Touch
$result->getDevice()->getBrand(); // Apple
$result->getDevice()->getType(); // portable media player

$resultArray = $result->toArray();
```
