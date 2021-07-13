---
title: "How to Configure a Magento Multistore in Docker"
date: 2020-08-04T10:51:22+02:00
draft: false

# meta description
description: "How to setup multiple websites, stores and store views in Magento locally with Docker using Warden and Traefik."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Magento 2

# post type
type: post
---
In warden it is possible to set up multiple domains for your project. Warden uses Traefik and is installed as a Global Service.

Traefik ensures that requests are intercepted and sent to the correct back-end service e.g. _https: //app.example1.test_ _https: //app.example2.test_. For this it uses an __HTTP reverse proxy__ and a __load balancer__.

It also has a dashboard to monitor different metrics. This can be found if you go to https: [https://traefik.warden.test/](https://traefik.warden.test/).

> A reverse proxy is a server that sits in front of web servers and forwards client (e.g. web browser) requests to those web servers.

{{<figure src="traefik-dashboard.png" alt="Traefik Dashboard" >}}

Before continuing, make sure you have set up a multistore in Magento. You can read how to do this in my previous blog post  [“How to Setup a Multistore in Magento 2”](https://rickdaalhuizen.com/posts/how-to-create-a-multi-store-in-magento/).

The following must be configured to set up a multi-domain locally for your Magento Shop.

 1. Sign certificates for your new domains
 2. Configure Nginx and Varnish to handle traffic coming from your new domains
 3. Configure Magento 2 to load different stores or websites by setting the run params.

## Step 1: Sign certificates for your new domains

```zsh
warden sign-certificate <your_new_domain>.test
```

This does the following.
 1. Update your ssl certificates
 2. Create a new ssl certificate.
 3. Update Traefik

This is the output
```zsh
❯ warden sign-certificate sportdrinks.test
==> Generating private key sportdrinks.test.key.pem
Generating RSA private key, 2048 bit long modulus
..+++
......................+++
e is 65537 (0x10001)
==> Generating signing req sportdrinks.test.crt.pem
==> Generating certificate sportdrinks.test.crt.pem
Signature ok
subject=/C=US/O=Warden.dev/CN=sportdrinks.test
Getting CA Private Key
==> Updating traefik
traefik is up-to-date
Connecting traefik to example_default network
Connecting tunnel to example_default network
Connecting mailhog to example_default network
Restarting traefik ... done
```

## Step 2: Configure Nginx and Varnish
From the root of your project you create a .warden/warden-env.yml file. This file exceeds the environment templates in warden. To see the complete configuration use: warden env config.


> Each environment's configuration uses a base configuration YAML file and optionally a darwin and linux file to add everything to the base configuration.

Add the following to your _.warden/warden-env.yml_

```yml
version: "3.5"
services:
  varnish:
    labels:
      - traefik.http.routers.${WARDEN_ENV_NAME}-varnish.rule=
          HostRegexp(`{subdomain:.+}.${TRAEFIK_DOMAIN}`)
          || HostRegexp(`{subdomain:.+}.<your_new_domain>.test`)
  nginx:
    labels:
      - traefik.http.routers.${WARDEN_ENV_NAME}-nginx.rule=
          HostRegexp(`{subdomain:.+}.${TRAEFIK_DOMAIN}`)
          || HostRegexp(`{subdomain:.+}.<your_new_domain>.test`)
```

Then run warden env up -d to update your containers.

## Step 3: Configure Magento 2 run params
Create an _app/etc/Stores.php_ and load it via composer.json according to the __[PSR-4 standard](https://www.php-fig.org/psr/psr-4/)__.

Create a file _app/etc/Stores.php_ with the following content:

```php
<?php

use \Magento\Store\Model\StoreManager;
$serverName = isset($_SERVER['HTTP_HOST']) ? $_SERVER['HTTP_HOST'] : null;

switch ($serverName) {
    case app.<your_new_domain>.test':
        $runCode = <code>;
        $runType = <type>;
        break;
}

if ((!isset($_SERVER[StoreManager::PARAM_RUN_TYPE])
        || !$_SERVER[StoreManager::PARAM_RUN_TYPE])
    && (!isset($_SERVER[StoreManager::PARAM_RUN_CODE])
        || !$_SERVER[StoreManager::PARAM_RUN_CODE])
) {
    $_SERVER[StoreManager::PARAM_RUN_CODE] = $runCode;
    $_SERVER[StoreManager::PARAM_RUN_TYPE] = $runType;
}
```

The run-code and run-time are the store code and type of your Magento Store or Website. You can find this under __Stores -> Settings -> All Stores__ or in one of the __store_ *__ tables.

```json
{
    "autoload": {
        "files": [
            "app/etc/stores.php"
        ]
    }
}
```

Run composer _dump-autoload_ to regenerate your autoload configuration.

Then restart warden containers and navigate to your new domain app. _<your_new_domain>.test_.
