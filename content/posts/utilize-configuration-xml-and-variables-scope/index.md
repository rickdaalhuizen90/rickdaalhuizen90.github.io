---
title: "Utilize configuration XML and variables scope."
date: 2020-09-04T17:48:38+02:00
draft: false

# meta description
description: "Magento 2 Exam Series - Magento Architecture & Customization Techniques: Determine how to use configuration files in Magento."

# featured image
feature: "banner.png"

# taxonomies
series:
- Magento Certification

tags:
 - Magento 2

# post type
type: post
---
Much of the configuration in Magento 2 is done by xml files, and placed in the _[Module_Root]/etc/_ directory.
Depending on the __scope__ of your configuration, place them in _etc/adminhtml_, _etc/frontend_ or under __global scope__ under _etc/_. When you put them in adminhtml or frontend, this overwrites the global scope.

## List of XML config files.
- _acl.xml_ - resource title, sort
- _adminhtml/rules/payment\_{country}.xml_ - paypal
- _address_formats.xml_
- _address_types.xml_ - format code and title only
- _cache.xml_ - name, instance - e.g. full_page=Page Cache
- _catalog_attributes.xml_ - catalog_category, catalog_product, unassignable, used_in_autogeneration, quote_item *
- _communication.xml_
- _config.xml_ - defaults
- _crontab.xml_ - group[], job instance, method, schedule *
- _cron_groups.xml_ *
- _di.xml_ - preference, plugins, virtual type *
- _eav_attributes.xml_ - locked entity attributes (global, unique etc.)
- _email_templates.xml_ - id label file type module -- view/frontend/email/name.html
- _events.xml_ - observers, shared, disabled *
- _export.xml_
- _extension_attributes.xml_ - for, attribute code, attribute type
- _fieldset.xml_
- _import.xml_
- _indexer.xml_ - class, view_id, title, description
- _integration.xml_
- _integration/api.xml_
- _integration/config.xml_
- _menu.xml_ - admin menu
- _module.xml_ - version, sequence
- _mview.xml_ - scheduled updates, subscribe to table changes, indexer model
- _page_types.xml_
- _payment.xml_ - groups, method allow_multiple_address
- _pdf.xml_ - renders by type (invoice, shipment, creditmemo) and product type
- _product_types.xml_ - label, model instance, index priority, (?) custom attributes, (!) composable types
- _product_options.xml_
- _resources.xml_
- _routes.xml_
- _sales.xml_ - collectors (quote, order, invoice, credit memo)
- _search_engine.xml_
- _search_request.xml_ - index, dimensions, queries, filters, aggregations, buckets
- _sections.xml_ - action route placeholder -> invalidate customer sections
- _system.xml_ - adminhtml config
- _validation.xml_ - entity, rules, constraints -> class
- _view.xml_ - vars by module
- _webapi.xml_ - route, method, service class and method, resources
- _widget.xml_ - class, email compatible, image, ttl (?), label, description, parameters
- _zip_codes.xml_ - [Add custom input mask for ZIP code](https://devdocs.magento.com/guides/v2.3/howdoi/checkout/checkout_zip.html)

## Configuration load order

The load order of the configuration files start with _app/etc/di.xml_.First it's collecting configration files under _[Module_Root]etc/*.xml_, then under _[Module_Root]/etc/[Area]/*.xml_, and finally merges them together.

While merging the configuration files, it is checked whether the id attribute of an xml node is unique, otherwise it will be overwritten with all underlying content (child nodes).

When the same xml document (configuration file) contains multiple nodes with the same id, it will give an error. -- [Magento DevDocs - Module configuration files](https://devdocs.magento.com/guides/v2.2/config-guide/config/config-files.html)

1. __Config merger class__
   - \Magento\Framework\Config\Dom
2. __When merging each file__
   - createConfigMerger|merge
   - \Magento\Framework\Config\Dom::_initDom(dom, perFileSchema)
   - \Magento\Framework\Config\Dom::validateDomDocument
   - $dom->schemaValidate
3. __After everything is merged__
   - \Magento\Framework\Config\Dom::validate(mergedSchema)
   - \Magento\Framework\Config\Dom::validateDomDocument

## Environment settings
Magento 2 comes with a configuration file _app/etc/env.php_ where you can store system-specific but also sensitive files, these can be API keys, passwords and personal information e.g. email or phone number. Sensitive data is not included during the export.

This file also contains your db data, crypt key to protect your passwords and other sensitive information (created during the Magento installation), backend frontName for your admin url and cache types.

To create a configuration you can do this as follows with _bin/magento_.

> To see more options run _bin/magento config_, options for __Magento 2.2.4 or higher__ may differ.

Set non-sensitive configuration
```bash
bin/magento config:set [--scope] [--scope-code] path value
```

Set sensitive configuration (writes to app/etc/env.php)
```bash
bin/magento config:sensitive:set [--scope] [--scope-code] path value
```

Show saved configuration
```bash
bin/magento config:show
```

Export configuration, let op: beschikt niet over gevoelige data!
```bash
bin/magento app:config:dump
```

Example
```bash
bin/magento config:set --scope=websites --scope-code=base web/unsecure/base_url http://example.com/
```

To find the right scope you can run the following sql query in your db:
```sql
SELECT * FROM store;
SELECT * FROM store_website;
```

With magerun:
```bash
n98-magerun.phar db:query "SELECT * FROM store;"
n98-magerun.phar db:query "SELECT * FROM store_website;"
```

For more information, you can consult the following Magento documentation
- [Set configuration values](https://devdocs.magento.com/guides/v2.2/config-guide/cli/config-cli-subcommands-config-mgmt-set.html)
- [Sensitive and system-specific](https://devdocs.magento.com/guides/v2.2/config-guide/prod/config-reference-sens.html)
- [Other configuration paths reference](https://devdocs.magento.com/guides/v2.2/config-guide/prod/config-reference-most.html)
