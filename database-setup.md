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

## Install MySQL client tools

1. Determine if you would prefer to interact with MySQL via the command line or via a locally installed GUI client. All of the tasks listed in the following sections can be accomplished via the GUI client or the command line. They are shown here as command line calls for simplicity. 
   1. If you prefer command line interaction, [download the MySQL community server](https://dev.mysql.com/downloads/mysql/) which matches your deployed MySQL version. You will not need to run the server itself, but you will need the client tools which are included in the server download.
      1. After download, extract the files in the download package
      2. Add the /bin directory under the extracted mysql directory to your system PATH
      3. Verify that you have access to the MySQL tooling on the command line:  `mysql --version`
   2. If you prefer UI-based interaction, [download the MySQL Workbench](https://dev.mysql.com/downloads/workbench/) which is a GUI-based MySQL administration tool. You may also need to download prerequisite packages, depending on your platform. After download, install the Workbench.

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
    create user 'accountsadmin'@'%' identified by 'accountsadminpassword';
    grant all privileges on duracloud_accounts.* to 'accountsadmin'@'%' ;
    create user 'accountsreader'@'%' identified by 'accountsreaderpassword';
    grant select on duracloud_accounts.* to 'accountsreader'@'%';
    create user 'milladmin'@'%' identified by 'milladminpassword';
    grant all privileges on duracloud_mill.* to 'milladmin'@'%';
    create user 'millreader'@'%' identified by 'millreaderpassword';
    grant select on duracloud_mill.* to 'millreader'@'%';
    flush privileges;
   ```
