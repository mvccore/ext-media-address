# MvcCore Extension - Router - Media

[![Latest Stable Version](https://img.shields.io/badge/Stable-v4.3.1-brightgreen.svg?style=plastic)](https://github.com/mvccore/ext-router-media/releases)
[![License](https://img.shields.io/badge/Licence-BSD-brightgreen.svg?style=plastic)](https://mvccore.github.io/docs/mvccore/4.0.0/LICENCE.md)
![PHP Version](https://img.shields.io/badge/PHP->=5.3-brightgreen.svg?style=plastic)

MvcCore Router extension to manage your website media version in URL
to have media flag in the request, controller and view to render different
templates, CSS and js files for mobiles, tablets or desktops.

## Outline  
1. [Installation](#user-content-1-installation)  
2. [Features](#user-content-2-features)  
	2.1. [Features - Routing](#user-content-21-features---routing)  
	2.2. [Features - Url Generating](#user-content-22-features---url-generating)  
3. [How It Works](#user-content-3-how-it-works)  
	3.1. [How It Works - Routing](#user-content-31-how-it-works---routing)  
	3.2. [How It Works - Url Completing](#user-content-32-how-it-works---url-completing)  
4. [Usage](#user-content-4-usage)  
	4.1. [Usage - `Bootstrap` Initialization](#user-content-41-usage---bootstrap-initialization)  
	4.2. [Usage - Media Url Prefixes And Allowed Media Versions](#user-content-42-usage---media-url-prefixes-and-allowed-media-versions)  
5. [Configuration](#user-content-5-configuration)  
	5.1. [Configuration - Session Expiration](#user-content-51-configuration---session-expiration)  
	5.2. [Configuration - Strict Session Mode](#user-content-52-configuration---strict-session-mode)  
	5.3. [Configuration - Routing `GET` Requests Only](#user-content-53-configuration---routing-get-requests-only)  

## 1. Installation
```shell
composer require mvccore/ext-router-media
```

[go to top](#user-content-outline)

## 2. Features

### 2.1. Features - Routing
- Router recognizes user device as three predefined versions - `full`, `tablet` or `mobile` - in first request by HTTP header `User-Agent` with third-party [`\Mobile_Detect`](https://github.com/serbanghita/Mobile-Detect) library.
- Router stores recognized device version string in its own session namespace with configurable expiration (to not process `\Mobile_Detect` recognition in every request again).
- Router completes `$request->GetMediaSiteVersion()` value to use it anywhere in your app as strings: `full`, `tablet` or `mobile`.
- Router replaces possibly founded media prefix substring in request path (`$request->GetPath()`) with an empty string. It keeps request path every time in the same form to process routing as usual.
- Strict mode media site version configuration option to drive application media version strictly by session value.

[go to top](#user-content-outline)

### 2.2. Features - Url Generating
- Router completes every application URL (or every `GET` URL by configuration) generated by built-in `Url()` method with media prefix substring by requested media version or media version given as second argument array with URL params for `Url()` method.
- There is also possible to configure the URL media version substring prefix.

[go to top](#user-content-outline)

## 3. How It Works

### 3.1. How It Works - Routing
- Router completes media site versions from these sources:
	- From requested URL (if there is no media site prefix in URL, it's completed to `full`).
	- From session (if there is nothing, it stays on `NULL`).
	- From special `$_GET` param to switch media site version in session strict mode (also could be `NULL`).
- Router process pre-route redirections by source data if necessary:
	- If there is allowed value in special `$_GET` switching param:
		  - New media site version is stored in session and request is redirected to new media site version by special switching param.
	- Else if there is no media site version in session from any previous request:
		  - There is recognized media site version by the third-party library [`\Mobile_Detect`](https://github.com/serbanghita/Mobile-Detect) and stored in the session for next requests.
		- There is also completed flag if the detected version is the same as the requested version.
	- If strict session mode is configured to `FALSE` (by default):
		- If the request is first (nothing is in session from previous requests):
			- If the detected version is different from the request version:
				- Redirect user to detected version:
			- Else route request with requested media site version in a standard way later, do not process any redirections.
		- Else route request with requested media site version in a standard way later, do not process any redirections.
	- If strict session mode is configured to `TRUE`:
		- If the requested media site version is different from the session version:
			- Redirect user to session version.
		- Else route request with requested media site version in a standard way later, do not process any redirections.
- Router removes any founded media site version URL prefix to process routing for any media site version with the same request path.
- Then the router, routes request in a standard way.

[go to top](#user-content-outline)
	
### 3.2. How It Works - Url Completing
- The router generates URL addresses always with the same media site version as requested media site version:
	- For addresses without any defined rewrite route, there is added into query string additional param about media site version (`&media_version=...`).
	- For addresses with defined rewrite route, there is prepended media site version URL prefix by router configuration.
- If requested version is `full` (`full` is by default), there is not necessary to put into URL addresses any additional data, so for `full` version, there is always the same original URL string without any special params or prefixes.
- If you define info build-in `Url()` method into second argument array into params any different media site version than requested media version is, there is added into result URL string query param or media site URL prefix by given media site version.
- if there is configured session strict mode, special `$_GET` switching param is always added automatically.

[go to top](#user-content-outline)

## 4. Usage

### 4.1. Usage - `Bootstrap` Initialization

Add this to `/App/Bootstrap.php` or to **very application beginning**, 
before application routing or any other extension configuration
using router for any purposes:

```php
$app = & \MvcCore\Application::GetInstance();
$app->SetRouterClass('\MvcCore\Ext\Routers\Media');
...
// to get router instance for next configuration:
/** @var $router \MvcCore\Ext\Routers\Media */
$router = & \MvcCore\Router::GetInstance();
```

[go to top](#user-content-outline)

### 4.2. Usage - Media Url Prefixes And Allowed Media Versions

There are configured three media site versions with URL address prefixes by default:
```php
use \MvcCore\Ext\Routers;
...
$router->SetAllowedSiteKeysAndUrlPrefixes([
    Routers\Media::MEDIA_VERSION_MOBILE	=> '/m',
    Routers\Media::MEDIA_VERSION_TABLET	=> '/t',
    Routers\Media::MEDIA_VERSION_FULL	=> '',
]);
```

To allow only selected media site versions and to configure url prefixes, you can use:
```php
// to allow only mobile version (with url prefix '/mobile') 
// and full version (with no url prefix):
use \MvcCore\Ext\Routers;
...
// now, tablet version is not allowed:
$router->SetAllowedSiteKeysAndUrlPrefixes([
    Routers\Media::MEDIA_VERSION_MOBILE    => '/mobile',
    // if you are using an empty string url prefix for full version, 
    // you need to define it as the last item!
    Routers\Media::MEDIA_VERSION_FULL    => '',
]);
```

[go to top](#user-content-outline)

## 5. Configuration

### 5.1. Configuration - Session Expiration
There is possible to change session expiration about detected media
site version value to not recognize media site version every request
where is no prefix in URL, because to process all regular expressions 
in `\Mobile_Detect` library could take some time. By **default** there is **1 hour**. 
You can change it by:
```php
$router->SetSessionExpirationSeconds(
    \MvcCore\Session::EXPIRATION_SECONDS_DAY
);
```

[go to top](#user-content-outline)

### 5.2. Configuration - Strict Session Mode
**In session strict mode, there is not possible to change media site version only by requesting different media site version prefix in URL.**
Stric session mode is router mode when media site version is managed by session value from the first request recognition. 
All requests to different media site version than the version in session are automatically redirected to media site version stored in the session.

Normally, there is possible to get different media site version only by 
requesting different media site version URL prefix. For example - to get 
a different version from `full` version, for example, to get `mobile` version, 
it's only necessary to request application with configured `mobile` prefix 
in URL like this: `/mobile/any/application/request/path`.

In session strict mode, there is possible to change media site version only by special `$_GET` parameter in your media version navigation. For example - 
to get a different version from `full` version, for example, `mobile` version, 
you need to add into query string parameters like this:
`/any/application/request/path?switch_media_version=mobile`
Then, there is changed media site version stored in the session and the user is redirected to the mobile application version with mobile URL prefixes everywhere.

To have this session strict mode, you only need to configure router by:
```php
$router->SetStricModeBySession(TRUE);
```

[go to top](#user-content-outline)

### 5.3. Configuration - Routing `GET` Requests Only
The router manages media site version only for `GET` requests. It means
redirections to the proper version in session strict mode or to redirect
in the first request to recognized media site version. `POST` requests
and other request methods to manage for media site version doesn't make sense. For those requests, you have still media site version record in session and you can use it any time. But to process all
request methods, you can configure the router to do so like this:
```php
$router->SetRouteGetRequestsOnly(FALSE);
```

[go to top](#user-content-outline)
