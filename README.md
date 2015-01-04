dbkit
=====

Tools to provision and prepare MSSQL db's for soak-it.


Usage example:
python -m dbkit.create -S mssql.somewhere.rds.amazonaws.com -U sa -P **** -d DEV_elvis_bingo -r sql-core -developer CORE

## Using the dbkit

The  **dbkit** is a general purpose thingie to allow you to do all kinds of db stuff f.e. create databases, update databases, get some info and so on.

All statements in examples below need to be excuted from the **vk-dbkit** folder.

Note that the command dbkit.cmd is doing "python -m dbkit.cli" behind the scenes.

 

## Creating all service databases for a tenant

Tenant needs to be registered in the admin.tenants table.

**dbkit.cli tenant createdb** is used to create all service databases for a tenant.  The following would create all service databases for tenant **elvis**...

 

dbkit -U login -P password tenant createdb elvis

 

## Creating a service database for tenant/service

Tenant needs to be registered in the admin.tenants table. Service needs to be registered as a multitenant service in the admin.services table.

**dbkit.cli tenant createdb** is also used to create one service databases for a tenant.  The following would create a **pilot** service database for tenant **elvis**...

 

dbkit -U login -P password tenant createdb elvis pilot

 

## Getting info using dbkit

It is possible to use dbkit to get info from the admin database.  The following is currently supported to get a list of services, tenants and databases...

 

dbkit -U login -P password service list

dbkit -U login -P password tenant list

dbkit -U login -P password database list

 

## Testing database create

Executing the following will register the tenant testYYYYMMDD and create all service databases for the tenant...

 

python -m dbkit.testcreate -U login -P password

Executing the following will register the tenant testYYYYMMDD so that a single database is used for all services (by setting admin.tenants.useSingleDb)...

 

python -m dbkit.testcreate -U login -P password -USESINGLEDB

Executing the following will register the tenant testYYYYMMDD so that service databases are created on a different DB instance (by setting admin.tenants.dbServer)...

 

python -m dbkit.testcreate -U login -P password -DBSERVER the_db_instance_name

Executing the following will drop all databases and delete rows from tables admin.databases and admin.tenants for tenant testYYYYMMDD, meaning its a cleanup that works for both multiple and single database tests regardless of where they are located...

 

python -m dbkit.testcreatecleanup -U login -P password

Note that you can also use **-t _tenant_** to use a tenant name of your own choice.

 

## Using dbkit for tenant add, edit, createdb, dropdb and remove

It is also possible to do all the steps done in testcreate manually one by one using tenant add, edit, createdb, dropdb and remove.

The following example will add the tenant mytesttenant, create all service databases for the tenant, drop all service databases for the tenant and remove the tenant...

 

dbkit -U login -P password tenant add mytesttenant

dbkit -U login -P password tenant createdb mytesttenant

dbkit -U login -P password tenant dropdb mytesttenant

dbkit -U login -P password tenant remove mytesttenant

The following example will add the tenant mytesttenant, create pilot and battle service databases for the tenant, drop pilot and battle service databases for the tenant and then remove the tenant...

 

dbkit -U login -P password tenant add mytesttenant

dbkit -U login -P password tenant createdb mytesttenant pilot

dbkit -U login -P password tenant createdb mytesttenant battle

dbkit -U login -P password tenant dropdb mytesttenant pilot

dbkit -U login -P password tenant dropdb mytesttenant battle

dbkit -U login -P password tenant remove mytesttenant

The following example will add the tenant mytesttenant, set description and useSingleDb for the tenant, create all service databases for the tenant, drop all service databases for the tenant and then remove the tenant...

 

dbkit -U login -P password tenant add mytesttenant

dbkit -U login -P password tenant edit mytesttenant description "some nice description text"

dbkit -U login -P password tenant edit mytesttenant usesingledb 1

dbkit -U login -P password tenant createdb mytesttenant

dbkit -U login -P password tenant dropdb mytesttenant

dbkit -U login -P password tenant remove mytesttenant

 

## Creating a database using dbkit.create directly

It is possible to use **dbkit.create** directly to create a database.  Note that this is not recommended as this will not read settings from the admin.tenants table and not automagically register the new database in the admin.databases table.

The following example shows how to create the **pilot** service database **vk_DEV_elvis_pilot** in such a direct non-recommended way...

 

python -m dbkit.create -S valkyrie-dev-1.ckgarvidcti5.eu-west-1.rds.amazonaws.com -U login -P password -d vk_DEV_elvis_pilot -r ..\vk-dbcore -developer VK-CORE

python -m dbkit.create -S valkyrie-dev-1.ckgarvidcti5.eu-west-1.rds.amazonaws.com -U login -P password -d vk_DEV_elvis_pilot -r ..\vk-pilot\vkpilot\db -developer VK-PILOT

 

## Static data Rx tables

All static data in Valkyrie is mastered in files under source control.  All static data has a globally unique **dbID** and **uniqueName** and most often also **displayName** and**description**.  References to static data in dynamic tables in the database are usually saved using **dbID** but the column name in the database table will be more specific f.e.**eventTypeID**, **shipClassID**, **loadoutID** and **upgradeID**.  All the static data is reflected in the general table **service.staticRx** (where the Rx stands for reflected).  So we have **admin.staticRx** in the vk_DEV_admin datbase, **pilot.staticRx** in the pilot databases and **battle.staticRx** in the battle databases.

Sometimes we will need to reflect more than just names and description meaning we will need to add more specific tables.  The tables **pilot.loadoutTypesRx** and**pilot.upgradeTypesRx** in the pilot databases are examples of such specific tables.

Currently the Dx tables can be viewed and synched using the service pages.

 

## Valkyrie reports on EVE Metrics

We currently have Valkyrie reports in the following groups on EVE Metrics...

- [Valkyrie](http://evemetrics/Reports?groupID=197)
- [VKDB](http://evemetrics/Reports?groupID=196)

You will find everything related to this on RESEARCHDB.  The job VALKYRIE: 02:00 executes the following...

 

EXEC vk_admin.admin.CopyDbStatsFromAmazon

EXEC vk_pilot.pilot.CopyDbStatsFromAmazon

EXEC vk_battle.battle.CopyDbStatsFromAmazon

EXEC vk_battle.battle.CopyBattleStatsFromAmazon

The CopyDbStatsFromAmazon procs copy DB stats from the vk_DEV databases on Amazon to the databases vk_admin, vk_pilot and vk_battle (tables zmetric.keyCounters and zsystem.lookupValues) and then the data proc data.DB_Stats_VK in the ebs_METRICS database generates all the DB stats reports in [VKDB ](http://evemetrics/Reports?groupID=196)from that data.

The CopyBattleStatsFromAmazon proc reads data from databases vk_DEV_default_battle and vk_DEV_playtest_battle and saves the [Valkyrie ](http://evemetrics/Reports?groupID=197)report data directly into the zmetric.keyCounters table in the ebs_METRICS database.

 
