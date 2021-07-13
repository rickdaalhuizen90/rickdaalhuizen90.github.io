---
title: "Moving from Sequel Pro to Sequel Ace"
date: 2020-09-11T17:48:26+01:00
draft: false

# meta description
description: "Sequel Ace is the \"Sequel\" for Sequel Pro, why and how to move from Sequel Pro to Sequel Ace."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Sequel Pro
 - Sequel Ace

# post type
type: post
---
Sequel Pro is an open-source SQL Client for MacOS, and for many Developers (myself included) the must-have tool when dealing with a SQL server. It's fast, has an easy interface, and is very intuitive to use.

You may have noticed that in recent years little has changed in terms of features and how it looks. After some recent updates in MacOS, I keep running into new bugs in Sequel Pro:

- [Sequel Pro Crashes in Mojave on closing a tab](https://github.com/sequelpro/sequelpro/issues/3360)
- [MacOS 10.15 Catalina Quick Connect & Crash](https://github.com/sequelpro/sequelpro/issues/3479)
- [Can't connect local database. Sequel Pro encountered an unexpected error](https://stackoverflow.com/questions/56759610/cant-connect-local-database-sequel-pro-encountered-an-unexpected-error)

It's also not surprising when you consider that the last release was [Sequel Pro - 1.1.2](https://sequelpro.com/news/post?id=1ee47439-a8ea-4937-86e9-be3216370889) on __April 3, 2016__ . Looking at the commit on their [Github page](https://github.com/sequelpro/sequelpro/graphs/contributors) it has only gotten less in recent years.

{{<figure src="github-commits.png" alt="Sequel Pro Github Contributtions" >}}

But now the good news! There is a replacement and that is [Sequel Ace](https://sequel-ace.com/), or whatever they call it: __"Sequel Ace is the" sequel "to longtime macOS tool Sequel Pro"__. In addition, it is open source and available on the [Apple App Store](https://apps.apple.com/us/app/sequel-ace/id1518036000?ls=1).

It is a fork of Sequel Pro so the interface remains largely the same, which makes switching to Sequel Ace even better.

## Preparation

- macOS >= 10.10
- MySQL >= 5.6
- MariaDB

## Installation

The easiest way is to install Sequel Ace via Homebrew

```bash
brew cask install sequel-ace
```

Now that it's installed you can open it with __⌘ + spacebar__ and type in __Sequel Ace__

{{<figure src="sequel-ace-startup.png" alt="Sequel Ace Startup Screen" >}}

## Import connections from Sequel Pro to Sequel Ace

The easiest way is to right click on export, and import this file back into Sequel Ace in the same way.

{{<figure src="sequel-pro-export.png" alt="Sequel Pro Export connections" >}}

Another way is to copy your files that are in the _~/Library_ read: [Migration from Sequel Pro to Sequel Ace](https://medium.com/@harrybailey/migration-from-sequel-pro-to-sequel-ace-c6a579399c90)

## Connecting to SSH

Before we want to make an SSH connection from Sequel Ace, we need to give it access first. This was not the case with Sequel Pro. The reason for this is that Sequel Pro runs in sandboxed modes. This also gives an extra layer of protection.

For this we need to add our _~/.ssh/config_ manually.

If you don't have one, make sure to create it first.

Here's an example of an ssh config for my local warden (Docker) environment.

```bash
Host tunnel.warden.test
  HostName 127.0.0.1
  User user
  Port 2222
  IdentityFile ~/.warden/tunnel/ssh_key
```

Now go to: Preferences -> Network -> SHH config: Other file ...

Select your _~/.ssh/config_ file.

Once you have added this you will see the file under: Preferences -> Files -> Available files.

When everything is setup right you can now connect to your SSH environment.

{{<figure src="sequel-ace-connection-succeeded.png" alt="Sequel Ace Connection Succeeded" >}}

Ps: Sequel Pro uses as default port: 22 for your SSH port.

## Conclusion

Hopefully, this has given you enough information on how to switch from Sequel Pro to Sequel Ace. If there are any questions or suggestions, let me know in the comments.