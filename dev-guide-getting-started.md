# Development Guide: Getting Started

The purpose of this guide is to walk you through the steps required to set up your local environment for working on the DuraCloud software. 

After you have completed these steps, you should have an operational DuraCloud deployment, running on your local machine.

## AWS account
If you don't already have an AWS account for working with DuraCloud, start by [creating one](https://aws.amazon.com/). While the DuraCloud software will be run on your local machine for development and testing, the software expects to be able to connect for AWS for email  ([SES](https://aws.amazon.com/ses/)), system notifications ([SNS](https://aws.amazon.com/sns/)), system queues ([SQS](https://aws.amazon.com/sqs/)), and storage ([S3](https://aws.amazon.com/s3/)).

Once you  have an AWS account, you will need to [add a user in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) with permissions to use SES, SNS, SQS, and S3. This user does not need to have console access, but access keys are required.

## Base software

Install the following software:

1. [Java (version 8+)](https://java.com/en/download/) - The Oracle JDK is recommended for building DuraCloud, as this JDK is used for release testing
2. [Maven (version 3.x)](https://maven.apache.org/download.cgi)
3. [Tomcat (version 8.x)](https://tomcat.apache.org/download-80.cgi)
4. [Git (latest version)](https://git-scm.com/downloads)

Once each of these is installed, open up your console (on Windows, consider using git-bash as your console, it's far more useful than the standard Windows command prompt) and verify that each is available on your path.
* For Java: `java -version`
* For Git: `git --version`
* For Maven: `mvn -version`
* For Tomcat: `version.sh`

If any of these fail to come back with a version value (usually saying something like "command not found"), you will need to add that program to your system path. How to update the PATH depends on your operating system, but [this may help](https://www.java.com/en/download/help/path.xml). In each application, you'll need to add the *bin* directory under the installation directory to the PATH

## Database

DuraCloud requires a MySQL database. You can [install MySQL Server](https://dev.mysql.com/downloads/) on your local machine or launch a server using [Amazon RDS](https://aws.amazon.com/rds/)

[Follow instructions](database-setup.md) to create the account and mill databases, create users, and grant the necessary permissions.

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
notification.user=<aws access key>
notification.pass=<aws password key>
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
JAVA_OPTS="${JAVA_OPTS} -Dduracloud.home<full path to duracloud home directory>"
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

## Build and Deploy

1. Check out the latest DuraCloud code:
```
git clone https://github.com/duracloud/duracloud.git
git clone https://github.com/duracloud/management-console.git
git clone https://github.com/duracloud/mill.git
```
2. Open your console, start Tomcat with: `startup.sh`
3. Open another console, `cd` into the management-console top directory and run:
```
mvn clean install
```
4. `cd` into the duracloud top directory and run:
```
mvn clean install -DskipIntTests
```
This will deploy the Management Console and DuraCloud applications into your local Tomcat

## Create an Account

**To be completed**

See: https://wiki.duraspace.org/display/DURACLOUDDOC/Deploying+the+Management+Console step #6 for the next steps to create an initial account
[The database script is here](https://raw.githubusercontent.com/duracloud/management-console/develop/resources/sql/make-user-root.sql)

## Contribute changes

* [Code Style](https://github.com/duraspace/codestyle)
* [Contribution Guidelines](code-guidelines.md)