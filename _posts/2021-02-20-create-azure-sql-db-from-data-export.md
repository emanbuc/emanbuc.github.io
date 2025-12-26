---
layout: post
title: Migrate an existing SQL database from local server to Azure SQL
date: 2021-03-05 12:10:00
description: There are specific tools you can use to migrate an existing database to Azure SQL.
tags: SQL Azure DB 
categories:  Azure DB 
#thumbnail: assets/img/azure_ml_01-02-workspace.png
---

# Create a New Azure SQL Database from Data Tier Application Export

If you want to migrate an existing SQL database from local server to Azure SQL  the old style backup and restore with *.back file can not be used with Azure SQL managed database. Also the newer Import/Export Data Tier application may fails depending on the PK /FK present into the tables.

There are specific tools you can use to migrate an existing database to Azure SQL.

- "Deploy to Azure" .
- "Import Database"
- Data Migration Assistant (DMA)

The first step is in both cases to create a backpack file with "Export Data Tier Application"


![export 1](/assets/img/sql_export_data_tier_01.png)

![export 1](/assets/img/sql_export_data_tier_01.png)

## Deploy database to Microsoft  Azure SQL

_If you can not connect to Azure from the original server you may restore a copy of the original DB in PC connected to Internet and the from here run "Deploy Database to Azure SQL"._

![Deploy to azure](/assets/img/creazione_db_azure_sql_04.png)

Follow the wizard and fill all required field

![Deployment settings ](/assets/img/creazione_db_azure_sql_05.png)

You need to connect to Azure SQL server with a user with privileges to create a new database such as a user with role "contributor" assigned in RBAC.

![Login to SQL server](/assets/img/creazione_db_azure_sql_01.png)

Please nota that you must select the appropriate authentication type from the drop-down list. Multi Factor Authentication (MFA) is active for may account so "Azure Active Directory" is the correct one from me.

Then the appropriate service plan for the new database. Nota that these settings can be changed later after deploy completed.

![Deployment settings - part 2 ](/assets/img/creazione_db_azure_sql_06.png)

Review deploy setting and click "Finish”.  When you do, you will see the progress of the migration, step by step while waiting until completed.

![Deployment settings - review ](/assets/img/creazione_db_azure_sql_07.png)

As you can see above, there are number of steps.  You can also see that there is a step named “Processing Table”.  This step will show each table as it is processed and migrated to Azure SQL Database.

![Deploying ](/assets/img/creazione_db_azure_sql_08.png)

## Import Database

TBC

![import database](/assets/img/azure_sql_import_01.png)

## Data Migration Assistant

This is a relatively easy tool to use.  It really has two purposes.

1. identify what challenges you might encounter when migrating at database
2. actually complete the migration.

This is a tool the must be [downloaded](https://www.microsoft.com/en-us/download/details.aspx?id=53595) and installed before using.   Once installed you will find it as _“Microsoft Data Migration Assistant”_ **in the Start menu, not under the SQL Server items**.

