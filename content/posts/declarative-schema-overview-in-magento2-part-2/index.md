---
title: "Declarative Schema Overview in Magento 2 - Part 2"
date: 2020-06-30T17:49:31+02:00
draft: false

# meta description
description: "How to migrate your Install/Upgrade scripts to declarative schema using safe mode and rollbacks in Magento 2."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Magento 2

# post type
type: post
---
In the first part we discussed what declarative schemes are, why you should use declarative schemes instead of install / upgrade scripts, how to create, edit and eventually delete a declarative. If you haven't already read the first part, I suggest you take a moment to read this first.
As an example, I created a simple module containing a InstallSchema.php that looks like this.

```php {linenos=inline}
<?php
/**
* Copyright (c) 2016 Magento. All rights reserved.
* See COPYING.txt for license details.
*/

namespace Rick\Example\Setup;

use Magento\Framework\DB\Ddl\Table;
use Magento\Framework\Setup\InstallSchemaInterface;
use Magento\Framework\Setup\ModuleContextInterface;
use Magento\Framework\Setup\SchemaSetupInterface;

/**
* @codeCoverageIgnore
*/
class InstallSchema implements InstallSchemaInterface
{
   /**
    * {@inheritdoc}
    * @SuppressWarnings(PHPMD.ExcessiveMethodLength)
    */
   public function install(SchemaSetupInterface $setup, ModuleContextInterface $context)
   {
       /**
        * Create table 'quote_item_file'
        */
       $table = $setup->getConnection()
           ->newTable($setup->getTable('quote_item_file'))
           ->addColumn(
               'entity_id',
               Table::TYPE_INTEGER,
               null,
               ['nullable' => false, 'primary' => true, 'identity' => true],
               'Entity ID'
           )
           ->addColumn(
               'filename',
               Table::TYPE_TEXT,
               255,
               ['nullable' => false],
               'Filename'
           )->addColumn(
               'location',
               Table::TYPE_TEXT,
               255,
               ['nullable' => false],
               'Location of file'
           )->addColumn(
               'quote_item_item_id',
               Table::TYPE_INTEGER,
               null,
               ['padding' => 10, 'nullable' => false, 'unsigned' => true],
               'Item Id'
           )->addForeignKey(
               $setup->getFkName(
                   $setup->getTable('quote_item_file'),
                   'quote_item_item_id',
                   'quote_item',
                   'item_id'
               ),
               'quote_item_item_id',
               $setup->getTable('quote_item'),
               'item_id',
               Table::ACTION_CASCADE
           )->setComment('Quote Item File Table');

       $setup->getConnection()->createTable($table);
   }
}
```

## Migrate install/upgrade scripts to a declarative scheme
Now it is also possible to convert existing installation / upgrade scripts to a declarative scheme with the "Schema Listener Tool". It is useful to know that using the Schema listener tool can only be used in developer mode, as it will make changes to the code. In addition, it does not take into account raw SQL data, and custom DDL operations outside of _\Magento\Framework\DB\Adapter\Pdo\Mysql_.

To perform the migration, run the following command:

```zsh
bin/magento s:up --convert-old-scripts=1
```

> For Magento versions less than 2.4 use the - convert_old_scripts flag

Note that this only works during an installation or upgrade of your modules. If you have issues generating the _db_schema.xml_, try removing the module entry in the setup_module table and then run setup:upgrade again.

This will generate a _db_schema.xml_ in your module etc directory.

```xml {linenos=inline}
<?xml version="1.0"?>
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
 <table name="quote_item_file" resource="default" engine="innodb" comment="Quote Item File Table">
   <column xsi:type="int" name="entity_id" padding="11" unsigned="false" nullable="false" identity="true" comment="Entity ID"/>
   <column xsi:type="varchar" name="filename" nullable="false" length="255" comment="Filename"/>
   <column xsi:type="varchar" name="location" nullable="false" length="255" comment="Location of file"/>
   <column xsi:type="int" name="quote_item_item_id" padding="10" unsigned="true" nullable="false" identity="false" comment="Item Id"/>
   <constraint xsi:type="primary" referenceId="PRIMARY">
     <column name="entity_id"/>
   </constraint>
   <constraint xsi:type="foreign" referenceId="QUOTE_ITEM_FILE_QUOTE_ITEM_ITEM_ID_QUOTE_ITEM_ITEM_ID" table="quote_item_file" column="quote_item_item_id" referenceTable="quote_item" referenceColumn="item_id" onDelete="CASCADE"/>
 </table>
</schema>
```

Then we add some data

```sql
INSERT INTO `magento`.`quote_item_file` (`entity_id`, `filename`, `location`, `quote_item_item_id`) VALUES ('1', 'examole.png', 'media/foo/bar/example.png', '1');
```

## What are schema whitelist and how to create them
Scheme whitelist is for backward compatibility. This keeps track of which adjustments have been made to the declarative schemes to prevent data loss, without finding out when this took place. When using an Install/Upgrade script this can often be found in the code.

The whitelist is generated in _<Module_Vendor>/<Module_Name>/etc/db_schema_whitelist.json_ and can be created as follows:

```zsh
bin/magento setup:db-declaration:generate-whitelist --module-name=<Module_Name>
```
And looks like this
```json
{
   "quote_item_file": {
       "column": {
           "entity_id": true,
           "filename": true,
           "location": true,
           "quote_item_item_id": true
       },
       "constraint": {
           "PRIMARY": true,
           "QUOTE_ITEM_FILE_QUOTE_ITEM_ITEM_ID_QUOTE_ITEM_ITEM_ID": true
       }
   }
}
```

> As a best practice, you should generate a new whitelist file for each release. You must generate the whitelist for any release that contains changes in the db_schema.xml file.

## Make use of dry-runs
Now we've created our _db_schema.xml_ and we want to alter the table in any way like adding a column, we can use dry-runs. Dry-runs assure that you don't have to worry about the script breaking something. No changes are made to the database during a dry run.

Let's remove one of the columns

```xml {linenos=inline, hl_lines=[6]}
<?xml version="1.0"?>
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
  <table name="quote_item_file" resource="default" engine="innodb" comment="Quote Item File Table">
    <column xsi:type="int" name="entity_id" padding="11" unsigned="false" nullable="false" identity="true" comment="Entity ID"/>
    <column xsi:type="varchar" name="filename" nullable="false" length="255" comment="Filename"/>
    <!-- <column xsi:type="varchar" name="location" nullable="false" length="255" comment="Location of file"/> -->
    <column xsi:type="int" name="quote_item_item_id" padding="10" unsigned="true" nullable="false" identity="false" comment="Item Id"/>
    <constraint xsi:type="primary" referenceId="PRIMARY">
      <column name="entity_id"/>
    </constraint>
    <constraint xsi:type="foreign" referenceId="QUOTE_ITEM_FILE_QUOTE_ITEM_ITEM_ID_QUOTE_ITEM_ITEM_ID" table="quote_item_file" column="quote_item_item_id" referenceTable="quote_item" referenceColumn="item_id" onDelete="CASCADE"/>
  </table>
</schema>
```

Run the following commands to use dry-runs:

```zsh
bin/magento s:up --dry-run=1
```

After you have performed the operation, a log file will be generated in _<Magento_Root>/var/log/ dry-run-installation.log_. This file contains raw SQL statements and can be used to optimize or modify your script where necessary.

```zsh
www-data@example-php-fpm:06:57 PM:/var/www/html$ cat var/log/dry-run-installation.log
ALTER TABLE `quote_item_file` DROP COLUMN `location`
```

## How to perform rollbacks
If for some reason something went wrong during the migration, a rollback can be used. For those who are not yet familiar with rollbacks, it ensures that the recent changes that have been made can be reversed.

Using the safe mode option will create a CSV file each time a destructive operation for a table or column occurs.

```zsh
bin/magento s:up --safe-mode=1
```

This will create a csv file under one of these locations:
 - _<Magento_root>/var/declarative_dumps_csv/{column_name_column_type_other_dimensions}.csv_
 - _<Magento_root>/var/declarative_dumps_csv/{table_name}.csv_

The csv file contains information from your database table, rows, columns, and values that have been modified for the installation.
In our example it looks like this

```zsh
www-data@example-php-fpm:07:40 PM:/var/www/html$ cat var/declarative_dumps_csv/quote_item_file_column_location.csv
entity_id,location
1,media/foo/bar/example.png
```

After verifying that everything is oke, we can remove the column from the table and run:

```zsh
bin/magento s:up
```

To revert everything back to the way it was, we just need to put the column we deleted back into _db_schema.xml_.

Finally we run the following to perform the rollback.

```zsh
bin/magento s:up --data-restore=1
```

> \-data-restore=1 is only possible with the setup:upgrade command

## Conclusion
I hope I've informed you enough about what declarative schemes in Magento are and how you can use them. I also recommend that you read the Magento documentation about "Migrate install/upgrade scripts to declarative schema" if you want to know more about it. For feedback or comments, please leave a message below.