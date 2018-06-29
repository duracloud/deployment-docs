# Development Guide: Getting Started

The purpose of this guide is to walk you through the steps required to set up your local environment for working on the DuraCloud software. 

After you have completed these steps, you should have an operational DuraCloud deployment, running on your local machine.

## AWS account
If you don't already have an AWS account for working with DuraCloud, start by [creating one](https://aws.amazon.com/). While the DuraCloud software will be run on your local machine for development and testing, the software expects to be able to connect to AWS for email  ([SES](https://aws.amazon.com/ses/)), system notifications ([SNS](https://aws.amazon.com/sns/)), system queues ([SQS](https://aws.amazon.com/sqs/)), and storage ([S3](https://aws.amazon.com/s3/)).

Once you  have an AWS account, you will need to:
* [Add a user in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) with permissions to use SES, SNS, SQS, and S3. This user does not need to have console access, but access keys are required.
* Verify an email address in the AWS SES service. In order to send emails via SES, the address used to send those emails needs to be verified. If SES is still in sandbox mode (where it starts for all new accounts) you will also need to verify addresses before sending mail to them. You can request that SES be transitioned into production mode at any time.
* Create a bucket in S3 for audit data. This bucket will be used to store audit logs. Using the default bucket settings will be fine. Capture the bucket name, you will need it later.
* Create a queue for audit data in the SQS service. The queue type should be Standard. This queue will be used to capture audit information as changes are made in DuraCloud. Capture the queue name, you will need it later.
* Create a topic in the SNS service. This topic will be used to send notifications from the Management Console to DuraCloud when users or accounts change. Capture the topic ARN, you will need it later.

## Base software

Install the following software:

1. [Java (version 8+)](http://www.oracle.com/technetwork/java/javase/downloads/index.html) - The Oracle JDK is recommended for building DuraCloud, as this JDK is used for release testing
2. [Maven (version 3.x)](https://maven.apache.org/download.cgi)
3. [Tomcat (version 8.x)](https://tomcat.apache.org/download-80.cgi)
4. [Git (latest version)](https://git-scm.com/downloads)

Once each of these is installed, open up your console (on Windows, consider using git-bash as your console, it's far more useful than the standard Windows command prompt) and verify that each is available on your path.
* For Java: `java -version`
* For Git: `git --version`
* For Maven: `mvn -version`
* For Tomcat: `version.sh`

If any of these fail to come back with a version value (usually saying something like "command not found"), you will need to add that program to your system PATH. How to update the PATH depends on your operating system, but [this may help](https://www.java.com/en/download/help/path.xml). In each application, you'll need to add the *bin* directory under the installation directory to the PATH.

Add a system environment variable named **JAVA_HOME** with the path to your Java JDK (the top level directory, not the /bin directory). This environment variable allows Tomcat and Maven to find Java.

## Database

DuraCloud requires a MySQL database. You can [install MySQL Server](https://dev.mysql.com/downloads/) on your local machine or launch a MySQL server using [Amazon RDS](https://aws.amazon.com/rds/)

[Follow the instructions at this link](database-setup.md) to create the account and mill databases, create users, and grant the necessary permissions.

## Configuration

### Create a  home directory
Create a directory on your file system to be used as a home directory for DuraCloud. Configuration files will be stored here, as well as logs and temporary data while the applications are running.

### Create application configuration file
In your DuraCloud home directory create a file named `duracloud-config.properties` with the following contents:
```
# Connection details for the account database
db.name=duracloud_accounts
db.host=<your-database-host>
db.port=3306
db.user=accountadmin
db.pass=<account admin password>

# Connection details for the mill database
mill.db.name=duracloud_mill
mill.db.host=<your-database-host>
mill.db.port=3306
mill.db.user=milladmin
mill.db.pass=<mill admin password>

# Deployment location for DuraCloud Management Console
mc.host=localhost
mc.port=8080
mc.context=ama
mc.domain=duracloud.org

# Connection details for AWS, to allow email notifications
notification.user=<aws iam access key id>
notification.pass=<aws iam secret key>
notification.from-address=<notification sender email address> 
notification.admin-address=<duracloud admin email address>
workDirectoryPath=/tmp/duracloud
```
Replace all values enclosed in `<>` with the correct values for your environment

### Make configuration available to DuraCloud applications
Create a file called `setenv.sh` in the `/bin` directory under your Tomcat installation with the following contents:
```
JAVA_OPTS="${JAVA_OPTS} -Dorg.duracloud.account.id=<account name>"
JAVA_OPTS="${JAVA_OPTS} -Dduracloud.config.file=file:<full path to duracloud-config.properties file>"
JAVA_OPTS="${JAVA_OPTS} -Dmc.config.file=file:<full path to duracloud-config.properties file>"
JAVA_OPTS="${JAVA_OPTS} -Dduracloud.home=<full path to duracloud home directory>"
JAVA_OPTS="${JAVA_OPTS} -Daws.accessKeyId=<aws iam access key id>"
JAVA_OPTS="${JAVA_OPTS} -Daws.secretKey=<aws iam secret key>"
```
Replace all values enclosed in `<>` with the correct values for your environment. The `<account name>` is the account name you will be creating in a later step. Go ahead and choose a name for this account now and add it to the configuration file.

### Configure Tomcat
*Configure Tomcat to allow for deployment of the DuraCloud web applications from your Maven build*

Edit /conf/tomcat-users.xml under your Tomcat installation, add this at the bottom (before the closing </tomcat-users> tag):

```
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="admin"/>
  <user username="[ANY-USERNAME]" password="[ANY-PASSWORD]" roles="admin,manager-gui,manager-script"/>
```
Replace the username and password values with new values (you'll need to make these up).

*Configure Tomcat to properly handle non-ASCII characters*

Edit /conf/server.xml under your Tomcat installation, add the config attribute "URIEncoding" with value "UTF-8" to your Tomcat Connector for port 8080.  Your connector should look like this when complete:
```
<Connector port="8080" protocol="HTTP/1.1" 
               connectionTimeout="20000" 
               redirectPort="8443"
               URIEncoding="UTF-8" />
```

### Configure Maven
*Configure Maven to be able to deploy to Tomcat as part of the build process*

Edit the /conf/settings.xml under your Maven installation, add the following within the `<servers>` tag:
```
  <server>
    <id>tomcat-server</id>
    <username>[ANY-USERNAME]</username>
    <password>[ANY-PASSWORD]</password>
  </server>
```
Make sure to use the same username and password you included in the Tomcat configuration above.

## Deploy the Management Console

1. Check out the latest Management Console code:
```
git clone https://github.com/duracloud/management-console.git
```
2. Open your console, start Tomcat with: `startup.sh`
3. Open another console, `cd` into the management-console top directory and run:
```
mvn clean install
```
This will deploy the Management Console application into your local Tomcat

## Configure the Management Console

1. Open a browser window and go to http://localhost:8080/ama. You should see the login page of the Management Console.
2. Select the *Create New Profile* link, fill out the form and click *Create Profile*. This will create the user account you will use to administer DuraCloud.
3. Connect to your MySQL database and execute the following script (make sure to first replace 'your-username-here' with your username). This will update your user to have root level access.
```
UPDATE duracloud_accounts.duracloud_user SET root=true WHERE username='your-username-here'; 
```
2. Back in the browser, login to the Management Console. In the upper right corner, select *Root Console*.
3. Select the *DuraCloud Mill* tab, then *Edit Configuration*. AWS configuration values you created in the AWS Account section at the top of this document will be needed here.
   1. Enter the Mill database information. The database name is `duracloud_mill`. The database port is 3306. The database username is `millreader`. Also enter the Audit queue name and space ID.
   2. Enter the audit queue name in the *Audit Queue* field and the S3 audit bucket name in the *Audit Log Space Id* field.
   3. Select *OK*.
4. Select the Global Properties tab, then *Edit Configuration*.
   1. Enter the SNS topic ARN you created in the AWS account section above
   2. For each of the CloudFront values enter any temporary value. These values will need to be updated later if there is a need to work with streaming capabilities.
   3. Select *OK*.

## Create an Account

1. In the Management Console, select Root Console, then the Accounts tab
2. Select the *Add Account* button
3. Name the test account anything you like, but ensure that the *Subdomain* field matches the value for the org.duracloud.account.id field in the setenv.sh file (which you created under the Tomcat /bin directory).
4. Choose *Amazon_S3* as the storage provider, and select the primary box. You can add additional providers later as needed.
5. Select *Finish*, then choose the *Configure Providers* button.
6. Enter the Access Key ID from the IAM User (that you created in the AWS account section above) into the *Username* field and the Secret Key into the *Password* field.
7. Select *Activate*. You now have an active DuraCloud account.

## Deploy DuraCloud Apps

1. Check out the latest DuraCloud code:
```
git clone https://github.com/duracloud/duracloud.git
```
2. Open a console, `cd` into the duracloud top directory and run:
```
mvn clean install -DskipIntTests
```
This will deploy the DuraCloud applications into your local Tomcat

3. Open a new browser window and go to http://localhost:8080. Log in with your root user credentials.

**Congratulations, you now have a functioning DuraCloud installation!!** :tada:

## Next Steps

Now that you have a functioning DuraCloud system, please work with us to make DuraCloud better! The following documents are helpful in describing how to contribute code:

* [Contribution Guidelines](code-guidelines.md)
* [Code Style](https://github.com/duraspace/codestyle)
