---
title: "How to Setup a Multistore in Magento 2"
date: 2020-07-28T13:39:31+02:00
draft: false

# meta description
description: "An easy guide of what multistores are, the hierarchy of a multistore and how you can create them in Magento 2."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Magento 2

# post type
type: post
---
It is possible to configure multiple instances in Magento.

Each instance can contain different attributes, including Languages, Domain names, Categories, Products, Currencies and more.

A Magento installation can have multiple websites that have multiple stores or store views.

## Magento 2 multistore hierarchy
__Global:__ At the top of the hierarchy. This is what you get when you install Magento. This includes all default configuration.

__Website:__ Magento installations begin with a single website which by default, is called Main Website. You can also set up multiple websites for a single installation, each with its own IP address and domain.

__Store:__ A single website can have multiple stores, each with its own main menu. The stores share the same product catalog, but can have a different selection of products and design. All stores under the same website share the same Admin and checkout.

__Store view:__ Each store that is available to customers is presented according to a specific view. Initially, a store has a single default view. Additional store views can be added to support different languages, or for other purposes. Customers can use the language chooser in the header to change the store view.

{{<figure src="magento2-multi-store-diagram.png" alt="Magento 2 multistore Diagram" >}}

## Step 1: How to create a multistore
After we have set up a Magento shop, we can log in as admin in our shop. If you don't have an admin account yet, you can create one as follows:

```zsh
bin/magento admin:user:create
```

As an example, let's take a website that sells Sport Drinks, as shown in the diagram above.

Go to Stores -> Settings -> All stores.

## Create a new Website
Click on Create Website and enter the following:

1. The name of your website
2. A unique code for your store
3. Sort is optional and defaults to 0, let's see how it is for now.

{{<figure src="magento2-create-new-website.png" alt="Magento 2 Create new Website" >}}

## Create a new store
Click on Create Store and enter the following:

1. Select website
2. Name of your Store
3. A unique code for your store
4. Root category, defaults to default

{{<figure src="magento2-create-new-store.png" alt="Magento 2 Create new Store" >}}

### Create a new store view
Click on Create Store and enter the following:

1. Select store
2. Name of your store view
3. A unique code for your store
4. Status enabled (self-explanatory)
5. Sort is optional and defaults to 0, let's see how it is for now.

{{<figure src="magento2-create-new-store-view.png" alt="Magento 2 Create new Store View" >}}

This is our result

{{<figure src="magento2-multistores.png" alt="Magento 2 multistores" >}}

## Method 1: Configure base urls from Admin
Next we need to set our base url in Stores> Settings> Configuration> General> Web.

1. Choose your store view at the top of the page for which you want to set your base url.
2. Complete the following fields
    1. Base URL: The baseurl of your store view.
    2. Base Link URL: The base url for your link urls, usually the same as your baseurl.
    3. Base URL for Static View Files (default value): Let's be default
    4. Base URL for User Media Files (default value): Enter URL for media files. This is often the baseurl with / media / after it. If you are using a CDN, the url is to your CDN.

{{<figure src="magento2-configure-base-url.png" alt="Magento 2 Configure Base Url" >}}

## Method 2: Configuring your base URL's using SSH

```zsh
bin/magento config:set web/unsecure/base_url "http://app.sportnutrition.test/" \
    --scope=website \
    --scope-code=sportnutrition \
    -le

bin/magento config:set web/secure/base_url "https://app.sportdrinks-nl.test/" \
    --scope=store \
    --scope-code=sportdrinks_nl \
    -le
```

## Configure the server to load the storefront
Now that we have created a new website for Magento we need to add it to our project. Make sure you are in developer mode.

```zsh
bin/magento deploy:mode:set developer
```

There are several ways you can do this.
 - Add the store code to your base URL, you don't need access to your server for this.
 - Create a subfolder with a .htaccess and index.php.
 - Using symlinks

## Method 1: Add the store code to your base URL.
Go to System -> Settings -> Configuration -> General -> Web
 1. Open the URL Options section:
 2. Choose Yes in the Add Store Code as you need.

URL with Store Code:
https://example.com/store-view/index.php/

URL without Store Code:
https://example.com/index.php/

## Method 2: Create a multistore using subfolder
Create a new folder in the root of your project and copy paste the .htaccess to your new folder.

```zsh
mkdir -p <new_site> && cp .htaccess <new_site>
```

Then you create an index.php:
```php
<?php
 
require realpath(__DIR__) . '/../app/bootstrap.php';
$params = $_SERVER;

$params[\Magento\Store\Model\StoreManager::PARAM_RUN_CODE] = '<code>';
$params[\Magento\Store\Model\StoreManager::PARAM_RUN_TYPE] = '<type>';
$bootstrap = \Magento\Framework\App\Bootstrap::create(BP, $params);

$app = $bootstrap->createApplication('Magento\Framework\App\Http');
$bootstrap->run($app);
```

## Method 3: Create a multistore with symlinks

Use _ln -s <source> <target_dir>_ to create a symlink.

```zsh
ln -s <path_to_projet_root>/app/ <your_new_site>app
ln -s <path_to_projet_root>/lib/ <your_new_site>lib
ln -s <path_to_projet_root>/pub/ <your_new_site>pub
ln -s <path_to_projet_root>/var/ <your_new_site>var
```

These were the steps to set up a multistore in Magento 2, in my next blog I will explain how you can configure your multistore locally with Docker and Warden.
