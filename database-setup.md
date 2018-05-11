# Database Setup
## Create your database server in RDS

1. Navigate to [RDS in the AWS console](https://console.aws.amazon.com/rds/home)
2. Select a mysql compatible database - choose your size according to your needs.
     For simplicity start with a db.m1.small, v. 5.6.x.
3. Select `no` for creating a replica in a different zone.
4. Enter your database name and root credentials.
5. Enter your preferences in terms of backup schedule, etc.
6. Do not put the database in a VPC at this time.
7. Create the instance
8. Navigate to the instance security groups, select the RDS group and add a rule with a CIDR/IP of 0.0.0.0/0

## Create account database
1. Clone the duracloud management console repository:
    ```git clone https://github.com/duracloud/management-console.git```
2. Checkout the latest tag via git. 
3. Connect to MySQL
    ```
    mysql -u <root-username> -h <your mysql host>
    ```
4. Create database  
    ```
    create database duracloud_accounts;
    exit;
    ```
5. Create database schema   
    ```
    mysql -u root -h your.mysql.host duracloud_accounts < management-console/resources/sql/schema-3.1.6.sql
    ```
## Create mill database
1. Clone the duracloud mill repository:  
    ```git clone https://github.com/duracloud/mill.git```
2. Checkout the latest tag via git
3. Connect to MYSQL
    ```
    mysql -u root -h <your mysql host>
    ```
4. create database  
    ```
    create database duracloud_mill;
    exit;
    ```
5. Create database schema   
    ```
    mysql -u root -h your.mysql.host duracloud_mill < mill/resources/mill-schema-2.5.2.sql
    ```
## Create database users and grant privileges  
1. Create 4 users in your database:

* accountsadmin - this user will have administrative rights to the accounts database
* accountsreader - this user will have read-only access to the accounts database
* milladmin - this user will have administrative rights to the mill database
* millreader - this user will have read-only access to the mill database

2. Grant rights to the users using these commands (**be sure to change the passwords in the script first**):

   ```
    grant all privileges on duracloud_accounts.* to 'accountsadmin'@'%' identified by 'accountsadminpassword';
    grant select on duracloud_accounts.* to 'accountsreader'@'%' identified by 'accountsreaderpassword';
    grant all privileges on duracloud_mill.* to 'milladmin'@'%' identified by 'milladminpassword';
    grant select on duracloud_mill.* to 'millreader'@'%' identified by 'millreaderpassword';
    flush privileges;
   ```
