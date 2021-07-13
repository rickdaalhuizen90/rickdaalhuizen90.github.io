---
title: "Declarative Schema Overview in Magento 2 - Part 1"
date: 2020-06-23T17:49:31+02:00
draft: false

# meta description
description: "What are declarative schema in Magento 2 and how to use them."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Magento 2

# post type
type: post
---
When we make changes to our database it's usually done through an installation or upgrade script. We start by creating an install script that has a php class "InstallSchema" and in it we write our PHP code to make adjustments to the database. Then when we need to change a table, we create an "UpgradeSchema", look at the version where it needs to upgrade and add our changes.

## The disadvantages of using installation scripts
During installation, Magento goes through all versions of the module until the latest version is reached. That looks like this:

```bash
1.0.0 create database schema (install table X)
1.0.1 update database schema (add column A to table X)
1.0.2 update database schema (add column B to table X)
1.0.3 update database schema (remove column A from table X)
```

While this all looks good at first, there are a number of drawbacks.

With this approach, the complexity of our upgrade script increases.

For example, it is not possible to delete the column you added earlier without increasing the module to a new version. Developers should fully understand what each installation and upgrade script contained without breaking anything. This results in added complexity and possible errors than necessary.

Fortunately, Magento came up with a solution in the form of a declarative scheme. The new approach allows developers to declare the ultimate desired state of the database and allows the system to automatically adjust to it without the need for redundant operations. In addition, there is no need to write an Uninstall script as the changes are automatically deleted when a module is removed.

## What are declarative schemas?
Declarative schemes have been introduced in Magento 2.3 and are a new way to make changes to the database. Using a declarative scheme is not yet a requirement in Magento 2.3 but it is recommended to use it. In addition to all the advantages that have already been mentioned above, it also has the following advantages:

 - dry-run mode for installation.
 - performance optimization: the declarative schedule approach saves significant time by moving from the current version to the last point.
 - supports rollbacks: developers can revert to a previous version.
 - validation before installation.

## How to use declarative schema
A declarative scheme comes in the form of an XML file. You can create this by adding a db_schema.xml file in the etc/folder.

For example, we want a table with the following information, a file name, the location of the file, and a relationship with a quote_item. We call the table "quote_item_file" with the following columns, filename, location, quote_item_item_id, created_at. This would look like this if we wrote it down in our etc/db_schema.xml.

```xml {linenos=inline, hl_lines=[6]}
<?xml version="1.0"?>
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
   <table name="quote_item_file" resource="default" engine="innodb" comment="Quote Item File Table">
       <column xsi:type="int" name="entity_id" padding="10" unsigned="true" nullable="false" identity="true"
               comment="Entity ID"/>
       <column xsi:type="varchar" name="filename" nullable="false" length="255"
               comment="Filename"/>
       <column xsi:type="varchar" name="location" nullable="false" length="255"
               comment="Location of file"/>
       <column xsi:type="int" name="quote_item_item_id" padding="10" unsigned="true" nullable="false"
               comment="Item Id"/>

       <!-- Our created_at column -->
       <column xsi:type="timestamp" name="created_at" on_update="false" nullable="true" default="CURRENT_TIMESTAMP"
               comment="Created At"/>

       <constraint xsi:type="primary" referenceId="PRIMARY">
           <column name="entity_id"/>
       </constraint>

       <constraint xsi:type="foreign" referenceId="QUOTE_ITEM_FILE_QUOTE_ITEM"
                    table="quote_item_file"
                    column="quote_item_item_id"
                    referenceTable="quote_item"
                    referenceColumn="entity_id" onDelete="CASCADE"/>
   </table>
</schema>
```
If you can see it is straightforward. We start with a table node with the following attributes, name, resource and engine. This again has the subnodes "column" that describe the columns of our table. In addition, it has a constraint subnode with the type “primary” for our Primary key and a constraint subnode with the type “foreign” when we want to create a relationship with another table, in our case the "quote_item” table.

Then we run:
```bash
bin/magento setup:db-declaration:generate-whitelist --module-name=Module_Name && bin/magento s:up
```

The db_scheme_whitelist.json serves to preserve the Backward compatibility in Magento.

When we go to our database you will see that our table has been created. If we want to change anything in our table, it is only a matter of modifying the file and running the commands again.

{{<figure src="magento-table.png" alt="Magento 2 Database table" >}}

To give an example of how easy this is, we add our created_at column to our table.

```xml
<?xml version="1.0"?>
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
  <table name="quote_item_file" resource="default" engine="innodb" comment="Quote Item File Table">
    <column xsi:type="int" name="entity_id" padding="11" unsigned="false" nullable="false" identity="true"
            comment="Entity ID"/>
    <column xsi:type="varchar" name="filename" nullable="false" length="255"
            comment="Filename"/>
    <column xsi:type="varchar" name="location" nullable="false" length="255"
            comment="Location of file"/>
    <column xsi:type="int" name="quote_item_item_id" padding="10" unsigned="true" nullable="false" identity="false"
            comment="Item Id"/>
         
    <constraint xsi:type="primary" referenceId="PRIMARY">
      <column name="entity_id"/>
    </constraint>
         
    <constraint xsi:type="foreign" referenceId="QUOTE_ITEM_FILE_QUOTE_ITEM_ITEM_ID_QUOTE_ITEM_ITEM_ID"
                table="quote_item_file"
                column="quote_item_item_id"
                referenceTable="quote_item"
                referenceColumn="item_id"
                onDelete="CASCADE"/>
  </table>
</schema>
```

Then we run _bin/magento s:up_ again, and see that the column has been added to the database.

{{<figure src="magento-table-2.png" alt="Magento 2 Database table 2" >}}

If we want to remove the table completely just delete it from the xml.

## Rename a table
It is possible to rename a table by creating a new table node and deleting the old one. Then we add the "onCreate" attribute to the new table node containing a method and the current table name as the argument. "migrateDataFromAnotherTable (quote_item_file)" see as follows.

{{<figure src="magento-git-diff.png" alt="Magento 2 Git diff" >}}


> When renaming a table, remember to regenerate the db_schema_whitelist.json file so it contains the new name in addition to the old one.

## Conclusion
So far the first part about declarative schemes in Magento 2. In the 2nd part I will discuss how you can convert Install/Upgrade scripts to a declarative scheme, dry-runs and how to perform rollbacks.
