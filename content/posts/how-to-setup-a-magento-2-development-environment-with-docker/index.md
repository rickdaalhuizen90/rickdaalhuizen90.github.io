---
title: "How to Setup a Magento 2 Development Environment with Docker"
date: 2020-07-21T14:32:47+02:00
draft: false

# meta description
description: "A complete guide on how to install an existing or clean Magento 2 development environment with docker and warden."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Docker
 - Warden
 - Linux

# post type
type: post
---
When I just started developing PHP applications I used Mamp or Xamp to set up a local Lamp stack.

Later this became Vagrant in combination with VirtualBox and eventually Docker.

The main advantage of Docker is portability, performance and it is scalable. This pays off, especially when you work in a team.

> When using commerce with Magento 2 it is possible to use the Magento Cloud Docker environment.

## Brief introduction to docker
The main difference between Docker and a VM is mainly the architecture between the 2. A VM is a computer software that mimics a real computer. For this he uses a hypervision — also called a "guest machine".

A hypervision can be a piece of software, firmware or hardware that the VM runs on. The hypervison itself runs on a real computer, this is called the "host machine".

The hypervison has a complete virtualization stack, such as network adapters, storage and CPU with its own operating system to run programs.

Containers and VM are similar, but the main difference between containers and a VM is that containers use the host computer kernel, which in turn shares it with other containers.

A container does not need a complete virtualization stack and its own os to run programs.

{{<figure src="vm-vs-docker.png" alt="VM vs Docker" >}}

However, if you want to delve deeper into the difference between docker and a VM, I recommend reading ["A Beginner-Friendly Introduction to Containers, VMs and Docker"](https://www.freecodecamp.org/news/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b/).

## What is Warden
For those who have already worked with Docker, you may recognize the time it can take to set up a working Docker environment. Especially if you don't have a DevOps team or sysadmin to set this up.

Now there are existing Docker environments for Magento 2 that you can find on Github, but my experience and that of my team was not always flawless. Often they were not up to date, they did not work as you would like, or you had to make adjustments yourself. Eventually we ended up with Warden.

Warden is a CLI utility that makes installing a Magento environment effortless even if you have little or no knowledge of Docker. According to the Warden documentation:

> Warden is a CLI utility for orchestrating Docker based developer environments, and enables multiple local environments to run simultaneously without port conflicts via the use of a few centrally run services for proxying requests into the correct environment’s containers.

This means that you can effortlessly run different local environments at the same time without conflicting with each other. Warden has the following features:

 - Traefik for SSL termination and routing/proxying requests into the correct containers.
 - Portainer for quick visibility into what’s running inside the local Docker host.
 - Dnsmasq to serve DNS responses for .test domains eliminating manual editing of /etc/hosts
 - An SSH tunnel for connecting from Sequel Pro or TablePlus into any one of multiple running database containers.
 - Warden issued wildcard SSL certificates for running https on all local development domains.
 - Full support for Magento 1, Magento 2, Laravel, Symfony 4, Shopware 6 on both macOS and Linux.
 - Ability to override, extend, or set-up completely custom environment definitions on a per-project basis.

## Preparations
To install Warden you first need Docker. Furthermore, it only runs on Linux or macOS - as far as I know there is no way to run warden on windows yet. If you are running macOS, I recommend installing [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac). For Linux, that's Docker for Linux. You also need docker compose and Mutagen 0.11.4 or higher.

## Install Docker
First make sure you have a package manager like Homebrew for macOS to install Docker Desktop for Mac, then run:
```bash
brew cask install docker
```

This installs the following: Docker Desktop, Docker Community Edition, Docker CE.

To install Docker for Linux you can use Docker engine.

With snapcraft:
```bash
sudo snap install docker
```

With apt-get:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
## Install warden

```bash
brew install davidalger/warden/warden
warden svc up
```

## Warden Global Services

Now that you have installed warden you can start warden by running:
```bash
warden up
```

This starts Warden's Global services. See _docker ps_
```bash
❯ warden up
Creating network "warden" with the default driver
Creating traefik   ... done
Creating portainer ... done
Creating tunnel    ... done
Creating dnsmasq   ... done
```
This includes traefik, portainer, dnsmasq and mailhog. This will give you access to the GUI and can be reached by going to the following urls:

 - [https://traefik.warden.test/](https://traefik.warden.test/)
 - [https://portainer.warden.test/](https://portainer.warden.test/)
 - [https://dnsmasq.warden.test/](https://dnsmasq.warden.test/)
 - [https://mailhog.warden.test/](https://mailhog.warden.test/)

## Warden commands

Warden has some useful commands that you can use to work with warden.

start warden:
```bash
warden up
```
stop warden:
```bash
warden down
```
start warden environment
```bash
warden env start
```
stop warden environment
```bash
warden env down
```
SSH in warden environment
```bash
warden shell
```
Run a command in Warden environment
```bash
warden env exec php-fpm bin/magento
```
Open MySQL session
```bash
warden db connect -A
```
Show PHP logs
```bash
warden env logs --tail 0 -f php-fpm php-debug
```

## Create a project

It only takes a few steps to set up a Warden environment for Magento 2.

Configure your Magento Marketplace credentials
```bash
composer global config http-basic.repo.magento.com <username> <password>
```

Let’s start by create a project 
```bash
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition --ignore-platform-reqs example && cd example/
```

Add a .env file to your project.
```bash
warden env-init <example> magento2
```
This creates an .env file in the root of your project. Want to customize your PHP version for example, you can do it here.

This .env file is used for the warden env commands. See warden env -h for more details.

## Create an SSL certificate

You can create an SSL certificate as follows:
```bash
warden sign-certificate example.test
```
Make sure the name matches “TRAEFIK_DOMAIN” which is in your .env

Now run:
```bash
warden env up -d
```
When you run this for the first time, the following are created:
 - Nginx
 - Varnish
 - PHP-FPM (7.0+)
 - MariaDB
 - Elasticsearch
 - RabbitMQ
 - Redis

\
This also starts the Mutagen sync session that syncs your files to your Docker container.

See result:
```bash
❯ warden env up -d
Creating network "example_default" with the default driver
Creating example_elasticsearch_1 ... done
Creating example_rabbitmq_1      ... done
Creating example_db_1            ... done
Creating example_redis_1         ... done
Creating example_mailhog_1       ... done
Creating example_php-fpm_1       ... done
Creating example_php-debug_1     ... done
Creating example_nginx_1         ... done
Creating example_varnish_1       ... done
Connecting traefik to example_default network
Connecting tunnel to example_default network
Starting example_redis_1         ... done
Starting example_elasticsearch_1 ... done
Starting example_db_1            ... done
Starting example_rabbitmq_1      ... done
Starting example_mailhog_1       ... done
Recreating example_php-fpm_1     ... done
Recreating example_php-debug_1   ... done
Recreating example_nginx_1       ... done
Recreating example_varnish_1     ... done
Created session sync_CQ4ZVwER7plFTM0pOaMGrFUnhopfXMcKksgfpPfNPJK
Waiting for initial synchronization to complete
...................................................................

```

> Files in the webroot are synced into the container using a Mutagen sync session with the exception of pub/media which remains mounted using a delegated mount.

Then run your _docker ps_ to see if everything is running.
```bash
❯ docker ps
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS              PORTS                                                 NAMES
230ac50b96e9        wardenenv/varnish:6.0                  "/bin/sh -c 'envsubs..."   2 minutes ago       Up 2 minutes        80/tcp                                                example_varnish_1
c366710da20c        wardenenv/nginx:1.16                   "/bin/sh -c 'envsubs..."   2 minutes ago       Up 2 minutes        80/tcp                                                example_nginx_1
706638d284aa        wardenenv/php-fpm:7.3-magento2-debug   "docker-entrypoint p..."   2 minutes ago       Up 2 minutes        9000/tcp                                              example_php-debug_1
06431cb0868a        wardenenv/php-fpm:7.3-magento2         "docker-entrypoint p..."   2 minutes ago       Up 2 minutes        9000/tcp                                              example_php-fpm_1
100e10dfc2a8        wardenenv/redis:5.0                    "docker-entrypoint.s..."   2 minutes ago       Up 2 minutes        6379/tcp                                              example_redis_1
1959774622b3        wardenenv/mariadb:10.3                 "docker-entrypoint.s..."   2 minutes ago       Up 2 minutes        3306/tcp                                              example_db_1
713ff0174084        wardenenv/mailhog:1.0                  "MailHog"                2 minutes ago       Up 2 minutes        1025/tcp, 8025/tcp                                    example_mailhog_1
9749c6aa8793        wardenenv/elasticsearch:6.8            "/usr/local/bin/dock..."   2 minutes ago       Up 2 minutes        9200/tcp, 9300/tcp                                    example_elasticsearch_1
b34f4a4c97ec        wardenenv/rabbitmq:3.7                 "docker-entrypoint.s..."   2 minutes ago       Up 2 minutes        4369/tcp, 5671-5672/tcp, 15671-15672/tcp, 25672/tcp   example_rabbitmq_1
29f678215b8c        traefik:2.1                            "/entrypoint.sh trae..."   19 hours ago        Up 2 minutes        127.0.0.1:80->80/tcp, 127.0.0.1:443->443/tcp          traefik
aa9fc5a1c1c1        portainer/portainer                    "/portainer"             19 hours ago        Up 19 hours         9000/tcp                                              portainer
58e4713d0345        jpillora/dnsmasq                       "webproc --config /e..."   19 hours ago        Up 19 hours         127.0.0.1:53->53/udp                                  dnsmasq
5687702db805        panubo/sshd:1.1.0                      "/entry.sh /usr/sbin..."   19 hours ago        Up 19 hours         127.0.0.1:2222->22/tcp                                tunnel
```

Now that the containers have been created and are running, we can proceed with the installation of Magento.

Open an ssh connection to your project
```bash
warden shell
```

## Install Magento

_Skip to the next step if you have an existing Magento project_

```bash
## Install Application
bin/magento setup:install \
    --backend-frontname=admin \
    --amqp-host=rabbitmq \
    --amqp-port=5672 \
    --amqp-user=guest \
    --amqp-password=guest \
    --db-host=db \
    --db-name=magento \
    --db-user=magento \
    --db-password=magento \
    --http-cache-hosts=varnish:80 \
    --session-save=redis \
    --session-save-redis-host=redis \
    --session-save-redis-port=6379 \
    --session-save-redis-db=2 \
    --session-save-redis-max-concurrency=20 \
    --cache-backend=redis \
    --cache-backend-redis-server=redis \
    --cache-backend-redis-db=0 \
    --cache-backend-redis-port=6379 \
    --page-cache=redis \
    --page-cache-redis-server=redis \
    --page-cache-redis-db=1 \
    --page-cache-redis-port=6379 \
    --elasticsearch-host=elasticsearch

## Configure Application
bin/magento config:set --lock-env web/unsecure/base_url \
    "https://${TRAEFIK_SUBDOMAIN}.${TRAEFIK_DOMAIN}/"

bin/magento config:set --lock-env web/secure/base_url \
    "https://${TRAEFIK_SUBDOMAIN}.${TRAEFIK_DOMAIN}/"

bin/magento config:set --lock-env web/secure/offloader_header X-Forwarded-Proto

bin/magento config:set --lock-env web/secure/use_in_frontend 1
bin/magento config:set --lock-env web/secure/use_in_adminhtml 1
bin/magento config:set --lock-env web/seo/use_rewrites 1

bin/magento config:set --lock-env system/full_page_cache/caching_application 2
bin/magento config:set --lock-env system/full_page_cache/ttl 604800

bin/magento config:set --lock-env catalog/search/engine elasticsearch7
bin/magento config:set --lock-env catalog/search/enable_eav_indexer 1
bin/magento config:set --lock-env catalog/search/elasticsearch7_server_hostname elasticsearch
bin/magento config:set --lock-env catalog/search/elasticsearch7_server_port 9200
bin/magento config:set --lock-env catalog/search/elasticsearch7_index_prefix magento2
bin/magento config:set --lock-env catalog/search/elasticsearch7_enable_auth 0
bin/magento config:set --lock-env catalog/search/elasticsearch7_server_timeout 15

bin/magento config:set --lock-env dev/static/sign 0

bin/magento deploy:mode:set -s developer
bin/magento cache:disable block_html full_page

bin/magento indexer:reindex
bin/magento cache:flush

## Generate an admin user
ADMIN_PASS="$(pwgen -n1 16)"
ADMIN_USER=localadmin

bin/magento admin:user:create \
    --admin-password="${ADMIN_PASS}" \
    --admin-user="${ADMIN_USER}" \
    --admin-firstname="Local" \
    --admin-lastname="Admin" \
    --admin-email="${ADMIN_USER}@example.com"
printf "u: %s\np: %s\n" "${ADMIN_USER}" "${ADMIN_PASS}"

```

## Install existing Magento 2 project

The following steps are the same as discussed above to the point of installing Magento. The first thing you need is a database export of your existing project, then import it as follows:
```bash
pv /path/to/<your_database_export>.sql | warden db import
```
Or copy your database export to your php-fpm container and use magerun with the drop tables option to import your database:
```bash
cp /path/to/<your_database_export>.sql <container_id>:/var/www/html
warden env exec php-fpm n98-magerun db:import --drop-tables <your_database_export>.sql
```

> Magerun is included by default when you install Warden, so you don't have to manually install it yourself.

When that is done you have to put the urls correctly in the _core_config_data_ table:

With Magerun:
```bash
warden env exec php-fpm n98-magerun config:set web/unsecure/baseurl  http://app.example.test/
warden env exec php-fpm n98-magerun config:set web/secure/baseurl  https://app.example.test/
warden env exec php-fpm n98-magerun web/unsecure/base_media_url http://app.example.test/media/
warden env exec php-fpm n98-magerun web/secure/base_media_url https://app.example.test/media/
```
Or you can open an interactive mysql session in your current project by:
```bash
warden db connect -A
```
And then by executing the following sql query:
```sql
UPDATE `core_config_data` SET value = 'http://app.example.test' WHERE path = 'web/unsecure/baseurl';
UPDATE `core_config_data` SET value = 'https://app.example.test' WHERE path = 'web/secure/baseurl';
UPDATE `core_config_data` SET value = 'http://app.example.test/media/' WHERE path = 'web/unsecure/base_media_url';
UPDATE `core_config_data` SET value = 'https://app.example.test/media/' WHERE path = 'web/secure/base_media_url';
```

Now that your installation is ready you can restart Warden.
```bash
warden env down && warden env up -d
```
Go to your application in your browser: [https://app.example.test](https://app.example.test)

{{<figure src="fresh-magento-2-install.png" alt="Fresh Magento 2 install" >}}

## Create an SSH connection to your database

Now that everything is up and running, you can connect to your database as follows.
 - Host: <WARDEN_ENV_NAME_db_1>
 - User: magento
 - Password: magento
 - Database: magento
 - SSH Host: tunnel.waren.test

##### Tableplus
Create a new connection > MySQL > Create

{{<figure src="tableplus.png" alt="Tableplus" >}}

##### Sequel Pro

{{<figure src="sequal-pro.png" alt="Sequel Pro" >}}

##### PHPStorm
1. Database > Data Source > MySQL
SSH/SSL > Use SSH tunnel > Select configuration >
\
{{<figure src="phpstorm-1.png" alt="PHPStorm SSH/SSL" >}}
2. SSH/SSL > Use SSH tunnel > Select configuration > 
\
{{<figure src="phpstorm-2.png" alt="PHPStorm Test connection 1" >}}
3. Test connection
\
{{<figure src="phpstorm-3.png" alt="PHPStorm Test connection 2" >}}
\
{{<figure src="phpstorm-4.png" alt="PHPStorm Test connection 3" >}}

## Install Grunt and configure Live reload

Check if grunt is installed:
```bash
❯ grunt --version
grunt-cli v1.3.2
```
If not:
```bash
npm install -g grunt-cli
```
Add the following to your _app/etc/env.php_
```php
<?php
return [
    'system' => [
        'default' => [
            'design' => [
                'footer' => [
                    'absolute_footer' => '<script src="/livereload.js?port=443"></script>'
                ]
            ]
        ]
    ]
];
```
Add your Gruntfile and package.json if you don't already have them.
```bash
cp Gruntfile.js.sample Gruntfile.js
cp package.json.sample package.json
```
Install node packages
```bash
npm install
```

Copy files to your Docker php-fpm container:
```bash
docker cp Gruntfile.js <container_id>:/var/www/html
docker cp package.json <container_id>:/var/www/html
docker cp node_modules <container_id>:/var/www/html
```

Perform the following:
```bash
warden env exec php-fpm grunt clean && grunt less
```
Finally
```bash
warden env exec php-fpm grunt watch
```
So far: "how to set up a local Docker environment for Magento with warden". In the 2nd part I will talk about how to set up multiple store views in Magento 2 and Warden.
