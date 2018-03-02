---
layout: post
title:  migrate redmine from 3.1.1 to 3.4.4
date:   2018-03-02 15:15:58
categories: Linux
tags: Linux
---


# migrate redmine from 3.1.1 to 3.4.4

first of all check the databases version,engine and character, make sure that are in same version engine and character.

check engine
~~~
SELECT @@default_storage_engine;
SELECT ENGINE FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'redmine';
show variables like 'char%';
~~~
check character
~~~
status
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
~~~
it looks like are utf8 character but actuly it not
~~~
show create table attachments
~~~
you will find that table's character still is latin1
~~~
SELECT *
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA="YourDataBaseName"
AND TABLE_TYPE="BASE TABLE";
~~~

my.cnf
```
[mysqld]
collation-server = utf8_unicode_ci
init-connect='SET NAMES utf8'
character-set-server = utf8
or
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
```

export database redmine_production
~~~
mysqldump　-t　redmine_production　-uroot　-p　>　redmine_production.sql　
~~~

before you import to new databases if not utf8
convert it to utf8
~~~
convert all tables character to another
SELECT CONCAT('ALTER TABLE `', TABLE_NAME,'` CONVERT TO CHARACTER SET utf8 COLLATE utf8_general_ci;') AS    mySQL
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA= "redmine_production"
AND TABLE_TYPE="BASE TABLE"
~~~

when you import data don't use
```
 mysql somedatabase < redmine_production.sql
```
just do this
~~~
mysql
use redmine_production
source /xxx/redmine_production.sql
~~~
there will be have some error that you cannot import. I found there have users and projects of table do not import to database

the old database of the redmine don't have fields of default_version_id and default_assigned_to_id, so when you import the table of projects you should specify the fields like:</br>
```
mysqldump redmind_product peojects > projects.sql
insert into table(fields1,fields2)value(xxx,xxx)
source /xxx/projects.sql
```
another one you should truncate </br>
```
mysqldump redmind_product users > users.sql
turncate users
source /xxx/users.sql
```
it's done for migrate databases
the rest of all is copy direcoty of files to new directory of files, you also have to install some plugins
I had installd plugins redmine_code_review and redmine_git_remote
~~~
apt remove imagemagick
apt update
apt install libmagickwand-dev libmysqlclient-dev libpq-dev
gem install rmagick mysql2 pg
~~~
download plugins
put it into direcoty of plugins run
~~~
rake redmine:plugins:migrate  RAILS_ENV=production
~~~
