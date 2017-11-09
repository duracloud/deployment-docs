# Database Setup
## Create your database server in RDS

1. Select a mysql compatible database - choose your size according to your needs.
     For simplicity start with a db.m1.small, v. 5.6.x.
1. Select no for creating a replica in a different zone.
1. Enter your database name and root credentials.
1. Enter your preferences in terms of back schedule, etc.
1. Do no put the database in a VPC at this time.
1. Create the instance

## Create account database
1. Clone the duracloud management console repository:  
    ```git clone https://github.com/duracloud/management-console.git```
1. Checkout the latest tag. 
1. ```mysql -u root -h <your mysql host>```
1. create database  
    ```
    create database duracloud_accounts;
    exit;
    ```
1. Create database schema   
    ```
    mysql -u root -h your.mysql.host duracloud_accounts < management-console/resources/sql/schema-3.1.6.sql
    ```
## Create mill database
1. Clone the duracloud mill repository:  
    ```git clone https://github.com/duracloud/mill.git```
1. Checkout the latest tag. 
1. ```mysql -u root -h <your mysql host>```
1. create database  
    ```
    create database duracloud_mill;
    exit;
    ```
1. Create database schema   
    ```
    mysql -u root -h your.mysql.host duracloud_mill < mill/resources/mill-schema-2.5.2.sql
    ```         
1. Create database users and grant privileges (be sure to change the passwords in the script first): 
    ```
    grant all privileges on duracloud_accounts.* to 'accountsadmin'@'%' identified by 'accountsadminpassword';
    grant select on duracloud_accounts.* to 'accountsreader'@'%' identified by 'accountsreaderpassword';
    grant all privileges on duracloud_mill.* to 'milladmin'@'%' identified by 'milladminpassword';
    grant select on duracloud_mill.* to 'millreader'@'%' identified by 'millreaderpassword';
    flush privileges;
    ```
