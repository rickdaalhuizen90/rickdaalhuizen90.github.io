---
title: "Magento 2 Module Development with Pestle"
date: 2020-06-16T17:49:31+02:00
draft: false

# meta description
description: "Use code generation with Pestle to automate your workflow in Magento 2."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Magento 2
 - Pestle

# post type
type: post
---

Magento 2 has a wide range of tools that help you with developing modules. One of the well-known CLI tools in Magento is [Magerun](https://github.com/netz98/n98-magerun2). This is an extension of Magento's own CLI tool that already comes out of the box when you install Magento 2. One of my favorite tools that I use a lot is [Pestle by Alan Storm](https://github.com/astorm/pestle).

Since much of our code we write mainly consists of template (reusable) code, Pestle helps to generate a lot of this code. Ultimately, this saves a lot of time when it comes to developing Magento 2 modules.

## What is Pestle
You can think of Pestle as a PHP framework that helps you build and organize CLI programs in Magento 2. For example, you can extend existing functions such as the "generate" function with your own template.
Who is familiar with Python, Pestle is similar to how Python imports modules.
It is also a collection of CLI programs with a focus on Magento 2 code generation.

## Install Pestle

Pestle comes in the form of a .phar file (PHP Archive file) and is easy to download as follows. Phar files are self-contained cross-platform, so it works on MacOS, Windows and Linux.

```bash
#with curl:
curl -LO http://pestle.pulsestorm.net/pestle.phar
#with wget:
wget http://pestle.pulsestorm.net/pestle.phar
```

Or you can download one of the other releases on their Github page.
Run the following "php pestle.phar version" to see which version is installed.

## What are we going to build?

We are going to build a simple module with a front-end page that we can navigate to. After installing pestle, run "php pestle.phar" to see a list of options.

We are particularly interested in the Magento 2 Generate options. To see only the "generate" options run:

```bash
php pestle.phar | grep generate --color=never
```

## Register a module
We start by registering a module. Now there are 2 options, the "full-module" and the "module" option.

The full module works slightly different from the module option. Instead of directly creating PHP and XML module files, this option generates a Unix shell script that executes other pestle commands to generate the module.

For now we stick to the "module" option. run:

```bash
php pestle.phar magento2:generate:module
```

Pestle will now come up with a few questions.
 - What the vendor namespace should be
 - What the name of the module should be.
 - Which version. See as follows:

```bash
www-data@example-php-fpm:04:51 PM:/var/www/html$ php pestle.phar magento2:generate:module
Vendor Namespace? (Pulsestorm)] Rick
Module Name? (Testbed)] Helloworld
Version? (0.0.1)] 
Creating [/var/www/html/app/code/Rick/Helloworld/etc] 
Created: /var/www/html/app/code/Rick/Helloworld/etc/module.xml
Created: /var/www/html/app/code/Rick/Helloworld/registration.php
```

After this is done you will see that under app/code/Rick a module HelloWorld has now been created. Run "bin/magento s:up" to register it.

{{<figure src="pestle-folder-structure.png" alt="Pestle folder structure" >}}

## Create a front-end route
Now that we have created and installed a module, we can create a route.

```bash
php pestle.phar magento2:generate:route
```

Then you fill in the questions just as you did with generate: module.

 - Choose in which module you want to create the route, in our case that is Rick_Helloworld
 - Choose whether it is a front-end or admin route, the default is front-end so just press enter.
 - The front-name and route id: rick_helloworld
 - Use default (Index) as "Controller name"
 - Use default (Index) as "Action name"


```bash
www-data@example-php-fpm:04:51 PM:/var/www/html$ php pestle.phar magento2:generate:route 
Which Module? (Pulsestorm_HelloWorld)] Rick_Helloworld
Which Area (frontend, adminhtml)? (frontend)] 
Frontname/Route ID? (pulsestorm_helloworld)] rick_helloworld
Controller name? (Index)] 
Action name? (Index)] 
Backing existing file: /var/www/html/app/code/Rick/Helloworld/etc/frontend/routes.xml.5f1db51e071db.bak.php
/var/www/html/app/code/Rick/Helloworld/etc/frontend/routes.xml
/var/www/html/app/code/Rick/Helloworld/Controller/Index/Index.php
```

After you will see a routes.xml in _app/code/Rick/HelloWorld/etc/routes.xml_

```xml {linenos=inline}
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="rick_helloworld" frontName="rick_helloworld">
            <module name="Rick_Helloworld"/>
        </route>
    </router>
</config>
```
If a routes.xml already exists, it will add a .bak extension, so that you keep a backup of your existing file. You can choose to remove it, if you no longer need it.

Finally, we create a front-end view by running:

```
php pestle.phar magento2:generate:view
```

Choose "Rick_Helloworld" as the module and accept the default values for Area, Handle, Block Name, Template File and Layout. Eventually you will see that pestle creates the following files.

{{<figure src="pestle-folder-structure-2.png" alt="Pestle folder structure 2" >}}

Finally delete your cache with "bin/magento c:c" and go to https: //app.example.test/Rick_helloworld/index/index

{{<figure src="pestle-result.png" alt="Pestle result" >}}

As you can see, within a short time I created, registered & installed a Magento 2 module, built a controller, built a route.xml with a block, layout and template without writing any code. Pestle offers a wide range of options that help you develop Magento 2 modules, so I definitely recommend you to look at the other options that Pestle has to offer.

## Useful commands in Pestle
In addition to generating code, it is also possible to perform scans and checks.

Corrects immediate use of PHP Object Manager

```bash
php pestle.phar magento2:fix-direct-om
```

Restores rights

```bash
php pestle.phar magento2:fix-permissions-mod
```
Scans modules for ACL rule IDs and ensures that they are all used / defined

```bash
php pestle.phar magento2:scan:acl-used
```

Searches for invalid keys in a MySQL database

```bash
php pestle.phar mysql:key-check
```

Migrating from Magento 1 code to Magento 2

```bash
php pestle.phar magento2:convert-class
php pestle.phar magento2:convert-observers-xml
php pestle.phar magento2:convert-system-xml
```