---
title: "Understanding Magento 2 Module-Based Architecture"
date: 2020-08-19T17:26:43+01:00
draft: false

# meta description
description: "Magento 2 Exam Series - Magento Architecture & Customization Techniques: What is a Magento 2 module and how does it work?"

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Magento 2

series:
 - Magento Certification

# post type
type: post
---

Let's start with what a module is, how a module works and how we can register it.

## Module overview

The Magento 2 documentation explains the following about modules:

> A module is a logical group – that is, a directory containing blocks, controllers, helpers, models – that are related to a specific business feature. In keeping with Magento’s commitment to optimal modularity, a module encapsulates one feature and has minimal dependencies on other modules. - [Magento DevDocs - Module overview]()

> Modules and themes are the units of customization in Magento. Modules provide business features, with supporting logic, while themes strongly influence user experience and storefront appearance. Both components have a life cycle that allows them to be installed, deleted, and disabled. From the perspective of both merchants and extension developers, modules are the central unit of Magento organization. - [Magento DevDocs - Module overview]()

> The Magento Framework provides a set of core logic: PHP code, libraries, and the basic functions that are inherited by the modules and other components. - [Magento DevDocs - Module overview]()

In short: a module is a logic group (folder) that contains business logic for a specific feature. A module can be installed in 2 places.

1. In vendor (when installed with composer): _vendor/[vendor]/[type]/[type]-[module-name]/[module-name]_.
2. In app/ (without composer):
	- as module: _app/code/[VendorName]/[ModuleName]_
	- as theme: _app/design/[type]/[VendorName]/[ThemeName]_
	- as language pack: _app/i18n/en_EN.csv_

When we talk about “type”, we are talking about a module, theme (admin or frontend themes) or language pack.

If a module contains a specific customization that is only related to a specific project, we can put it in _app/code/[vendor]/[type]-[module-name]_.

It's common to develop our module first in app/ and move it into vendor when a first "stable" release is ready.


## Module registration
Magento components, including modules, themes, and language packages, must be registered in the Magento system through the Magento ComponentRegistrar class. - [Magento DevDocs - Register your component]()

When we build a module, it must be registered in order to be recognized by Magento. This is done by means of the Magento [ComponentRegistrar class](https://github.com/magento/magento2/blob/2.4-develop/lib/internal/Magento/Framework/Component/ComponentRegistrar.php) in _registration.php_. You create this in the root of your module.

- Register a module

```php
<?php
use \Magento\Framework\Component\ComponentRegistrar;
ComponentRegistrar::register(ComponentRegistrar::MODULE, '[Vendor_ModuleName]', __DIR__);
```

- Register a language pack
```php
<?php
use \Magento\Framework\Component\ComponentRegistrar;
ComponentRegistrar::register(ComponentRegistrar::LANGUAGE, '[Vendor_PackageName], __DIR__);
```

- Register a Frontend or Admin theme, where _[area]_ is frontend or adminhtml.

```php
<?php
use \Magento\Framework\Component\ComponentRegistrar;
ComponentRegistrar::register(ComponentRegistrar::THEME, '[area]/[vendor]/[theme-name], __DIR__);
```

Then we have to autoload our registration.php in /composer.json (root of your module).
```json
{
    "name": [vendor]/[module-name]
    "autoload" {
        "psr-4": { "[Vendor]\\[ModuleName]\\": "" },
        "files": ["registration.php"]
    }
}
```

After we've created our registration.php and added this to our composer autoload, we need to create a _/etc/module.xml_.
It basically tells Magento that it exsist.

The sequence nodes describes the order that the module has to load. This might be usefull when you're working with plugins, preferences or other dependencies such as layouts.

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="YourCompany_YourModule" setup_version="1.0.0">
        <sequence>
            <module name="TheirCompany_TheirModule" />
            <module name="AnotherCompany_AnotherModule" />
        </sequence>
    </module>
</config>
```

## Modules interaction
The way modules in Magento 2 interact with each other is by means of dependency injection, [Service Contracts]() or [Data Service Contracts](). An important aspect of interaction is the scope in which the component is located. In other words, a Magento area.

Some benefits of using service contracts is that these contracts ensure a well-defined, durable API that other modules and third-party extensions can implement. Also, these contracts make it easy to configure services as web APIs.

Magento docs: [Service contract anatomy](https://devdocs.magento.com/guides/v2.2/architecture/archi_perspectives/service_layer.html#service-contract-anatomy)

## Modules limitation and side effects
However, there're some limitations and should be considered when you're working with modules in Magento 2.
You should use depends to indicate that a module depends on another module - [Module dependencies](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_depend.html)

1. A module is responsible for 1 feature.
2. A module that is dependent on other modules must be explicitly stated in _etc/modules.xml_, using depends.
3. When a module is installed or removed it should have no effect on other modules.

Some common errors are:
- [Missing Class ReflectionException](https://laracasts.com/discuss/channels/laravel/reflectionexception-class-not-found-on-repository)
- [Class MissingClass does not exist](https://stackoverflow.com/questions/31364289/error-class-does-not-exist)
- [PHP Warning: Uncaught Error: Class MissingClass not found](https://dev.to/dechamp/php---how-to-fix-class--not-found-error-1gp9)

## Magento areas
A Magento area only loads the components that are related to that area. When a component is loaded, you only need to look at the area where the request takes place. This ensures a more optimized process. In addition, it ensures that a component cannot influence different areas.

> You can enable or disable an area within a module. If this module is enabled, it injects an area’s routers into the general application’s routing process. If this module is disabled, Magento will not load an area’s routers and, as a result, an area’s resources and specific functionality are not available.
-- [Magento DevDocs - Modules and areas]()

## Magento area types
1. __Adminhtml__ - Any code you see in the admin panel
2. __Frontend__ - The storefront (or frontend) has all the template and layout files that you see on the front.
3. __Base__ - This is the fallback when there is no adminhtml or frontend area.
4. __Crontab__ - Located in cron.php and loaded with _\Magento\Framework\App\Cron_.
5. __webapi_rest__ - Used for REST APIs
6. __webapi_soap__ - Used for SOAP APIs

To see how areas work, you can find the code [Magento/Framework/App/Area.php](https://github.com/magento/magento2/blob/2.0/lib/internal/Magento/Framework/App/Area.php)
