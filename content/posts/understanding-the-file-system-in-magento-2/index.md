---
title: "Understanding the file system in Magento 2"
date: 2020-08-24T17:48:38+02:00
draft: false

# meta description
description: "Magento 2 Exam Series - Magento Architecture & Customization Techniques: Determine how to locate different types of files in Magento."

# featured image
feature: "banner.png"

# taxonomies
tags:
- Magento 2

series:
- Magento Certification
---

> One of the first things you can do to get started with component development is to understand and set up the file system. Each type of component has a different file structure, although all components require certain files. In addition, you can choose the component root directory to start development. The following sections have more information. - [Magento DevDocs - About component file structure]()

## Where to find Javascript and PHTML files?

- view/frontend/web/js
- view/frontend/requirejs-config.js
- view/frontend/layout
- view/frontend/templates

## Component root directory
A component root directory can take place in 2 places, /app and /vendor, and it's the __top-level directory__ for that component.

1. __/app__
	- Modules - _app/code_.
    - Storefront themes - _app/design/frontend_.
    - Admin themes - _app/design/adminhtml_.
    - Language packages - _app/i18n_.
2. __/vendor__
	- Modules - _vendor/[vendor-name]/[module-module-name]_
	- Storefront themes - _vendor/[vendor]/theme-[area]-[theme-name]_
	- Admin themes - _vendor/[vendor-name]/theme-adminhtml-[theme-name]_
	- Language packages - _vendor/[vendor-name]/language-en_us_

> Any third party components (and the Magento application itself) are downloaded and stored under the vendor directory. If you are using Git to manage a project, this directory is typically added to the .gitignore file. Therefore, we recommend you do your customization work in app/code, not vendor.
-- [Magento DevDocs - Create your component file structure]()

## Component file structure
A component requires a registration.php, etc/module.xml and a composer.json file. That's enough for Magento 2 to know that a component exsist. See my previous blog about [Understanding Magento 2 Modules-Bsed architecture - Module registration](http://localhost:1313/posts/understanding-magento-2-module-based-architecture/#module-registration)

The following files and folders are common to see in Magento 2 Module (Component).

- __Api__ - Any PHP classes exposed to the API (Service and Data Service contracts).
- __Block__ - PHP view classes as part of Model View Controller(MVC) vertical implementation of module logic.
- __Console__ - Console commands
- __Controller__ - PHP controller classes as part of MVC vertical implementation of module logic.
- __Controller/Adminhtml__ - Admin controllers
- __Cron__ - Cron job classes
- __etc__ - Configuration files; in particular, module.xml, which is required.
- __Helper__ - Helpers
- __i18n__ - Localization files in CSV format
- __Model__ - PHP model classes as part of MVC vertical implementation of module logic.
- __Model/ResourceModel__ - Database interactions
- __Observer__ - Event listeners
- __Plugin__ - Contains any needed plug-ins.
- __Setup__ - Classes for module database structure and data setup which are invoked when installing or upgrading.
- __Test__ - Unit tests
- __Ui__ - UI component classes
- __view__ - View files, including static view files, design templates, email templates, and layout files.
	- view/{area}/email
	- view/{area}/layout
	- view/{area}/templates
	- view/{area}/ui_component
	- view/{area}/ui_component/templates
	- view/{area}/web
	- view/{area}/web/template
	- view/{area}/requirejs-config.js

## Theme file structure
A Magento theme is usually located in _app/design/frontend/[Vendor]/_, or in _vendor/[vendor]/theme-[area]-[theme-name]_ but can technically be placed anywhere. However, a theme must be placed in a separate folder.

A typical magento theme folder structure looks like this:

```bash
[theme_dir]/
├── [Vendor]_[Module]/
│   ├── web/
│   │   ├── css/
│   │   │   ├── source/
│   ├── layout/
│   │   ├── override/
│   ├── templates/
├── etc/
├── i18n/
├── media/
├── web/
│   ├── css/
│   │   ├── source/
│   ├── fonts/
│   ├── images/
│   ├── js/
├── composer.json
├── registration.php
├── theme.xml
```

To create a theme you basically only need the following, these are required to create a theme.
 - __Composer.json__ - Describes the theme dependencies and metadata. Is required if your theme is a composer package.
 - __Registration.php__ - Is required to register your theme
 - __Theme.xml__ - It contains the basic meta-information, like the theme title and the parent theme name

The following folders are optional:

 - __layout/__ - Layout files which extend the default module or parent theme layouts. 
 - __layout/override/base__ - Layouts that override the default module layouts. 
 - __layout/override/<parent_theme>__ - Layouts that override the parent theme layouts for the module.
 - __templates__ - This directory contains theme templates which override the default module templates or parent theme templates for this module. Custom templates are also stored in this directory. 
 - __etc/view.xml__ - This file contains configuration for all storefront product images and thumbnails. 
 - __i18n__ - .csv files with translations.
 - __media__ - This directory contains a theme preview (a screenshot of your theme). 
- __web__ - Static files that can be loaded directly from the frontend.
	- web/fonts - Theme fonts. 
	- web/images - Images that are used in this theme. 
	- web/js - Theme JavaScript files.
	- web/css - Theme Less files
		- web/css/source - This directory contains theme less configuration files that invoke mixins for global elements from the Magento UI library, and theme.less file which overrides the default variables values. 
		- web/css/source/lib - View files that override the UI library files stored in lib/web/css/source/lib

<!-- ### What are the naming conventions, and how are namespaces established?

### How can you identify the files responsible for some functionality? -->