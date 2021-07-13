---
title: "Setting up PhpStorm for Magento 2"
date: 2020-07-07T17:48:38+02:00
draft: false

# meta description
description: "Step by step guide on how to setup PhpStorm for Magento 2 development, and how to configure Code Sniffer, Mess detector and much more."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Magento 2
 - PhpStorm

# post type
type: post
---
As a Magento developer you are dealing with a complex code-base that you have to navigate through. That is why it is useful to have an IDE (Integrated Development Environment) that helps you write your code and ensure its quality.

Perhaps one of the most famous IDE for PHP developers is PHPStorm. PHPStorm comes out of the box with all the tools you need for writing PHP applications. Here are some tips and tricks about my PHPStorm configurations that I use as a Magento Developer on a daily basis.

I will not cover the following: keyboard shortcuts, fonts, and color schemes, since they will be different for everyone. Eventually I will share my settings.zip with you for download.
Go to the menu bar and hide what you don't need:

## Show options menu
✔ Compact directories (Ensures less nesting of empty folders)

## View > Appearance
Hide the following:\
☐ Tool Window Bars\
☐ Status Bar\
☐ Navigation bar

Then go to Phpstorm -> Preferences or open "Preferences" with the key combination ⌘ + ,
## Appearance & Behavior > Appearance
☐ Show tool window numbers\
☐ Show tool window bars\
☐ Enable mnemonics in menu

## Appearance & Behavior > System Settings
☐ Reopen projects on startup\
☐ Confirm before exiting the IDE\
✔ Open project in new window

## Editor > General > Breadcrumbs
☐ Show Breadcrumbs

## Editor > General
✔ Ensure an empty line at the end of a file on Save

## Directories:
Exclude files:
```bash
.github;.idea;generated;phpserver;pub;setup;update;var
```
This gives us a more organized folder structure with only the folders we care about.

{{<figure src="phpstorm-exclude-dir.png" alt="PHPStorm exclude dir" >}}

## Plugins
I like to keep it as minimal as possible by only installing essential plugins, these are as follows.
 - __IdeaVim__ - Vim emulation plug-in for IDEs based on the IntelliJ platform
 - __Magento PhpStorm__ - Official Magento PhpStorm plugin for Magento 2
 - __MaGinto - Plugin__, created to improve life-work balance while working with Magento 2
 - __PHP Annotation__ - Extends PhpStorm to support annotations in DocBlocks
 - __Symfony Support__ - Support for Symfony framework / components.
 - __BashSupport__ - Adds extra support for bash scripts
 - __CSV Plugin__ - Lightweight plugin for editing CSV/TSV/PSV files
 - __.​env files support__ - easy support for dotenv files
 - __Solarized Themes__ - Adds Solarized Dark and Light color themes.

{{<figure src="phpstorm-plugins.png" alt="PHPStorm Plugins" >}}

## Apply PSR1/PSR2 Standard to your code
⌘ + , Code Style -> PHP -> Set from… -> PSR1/PSR2

{{<figure src="phpstorm-codestyle.png" alt="PHPStorm Codestyle" >}}

## Configure PHP Codesniffer (phpcs), PHP Mess detector (phpms) and PHP CS Fixer
⌘ +, Language & Framework -> PHP -> Quality tools -> PHP CodeSniffer -> …
 - PHP_CodeSniffer path: <Magento_Root>/vendor/bin/phpcs
 - Path to phpcbf: <Magento_Root>/vendor/bin/phpcbf

Click the "Validate" button to make sure everything is set-up right.
Repeat the same steps for Mess Detector and PHP CS Fixer. You can find these files under: <Magento_Root>/vendor/bin/

{{<figure src="phpstorm-codesniffer.png" alt="PHPStorm CodeSniffer" >}}

And lastly, don't forget to backup your configuration by going to Files -> Manage IDE Settings -> Export Settings, you can download mine in here.
This is the end of my blog, if I missed something or you have any settings you want to share, please leave a comment below .