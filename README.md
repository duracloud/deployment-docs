# DuraCloud Deployment Documentation

The following documents capture the steps required to build, release, and deploy DuraCloud software

### Initial setup
* [General AWS Environment Setup](aws-setup.md) - Describes the initial setup of an AWS account and the services which support DuraCloud deployment
* [Database setup](database-setup.md) - Describes the setup of Amazon RDS to support DuraCloud
* [Mill deployment] (deploy-mill.md) - Defines the steps required to deploy and configure the DuraCloud Mill
* [DuraCloud Application](duracloud-webapp-setup.md) - Describes the process of deploying the DuraCloud applications (DuraStore and DurAdmin) using AWS Elastic Beanstalk
* [Management Console](management-console-setup.md) -  Describes the process of deploying the DuraCloud Management Console using AWS Elastic Beanstalk
* [DNS mapping and management]() - Describes configuration of DNS to support DuraCloud
* [Monitoring tools]() - Describes how to setup monitoring tools used with DuraCloud
* [Logging config]() - Describes log capture and visualization using SumoLogic
* [DuraCloud Bridge]() - Describes configuration of the DuraCloud Bridge application, coordinating this with the DuraCloud applications, and use of the Bridge API

### System Updates
* [Database updates]() - Describes how to perform database updates
* [Application updates]() - Describes how to update the DuraCloud applications and Management Console software in Elastic Beanstalk
* [Mill updates]() - Describes how to update the DuraCloud Mill software and configuration

### Account Management
* [Creating new accounts](creating-new-accounts.md) - Describes how a new client DuraCloud account is added to the system

### Architecture
* [DuraCloud Architecture]() - Describes the overall DuraCloud architecture
* [Database interaction]() - Describes how DuraCloud applications interact with the database
* [Storage Providers]() - Describes how DuraCloud interacts with storage

### Software Development
* [DuraCloud code contribution guidelines]() - Decribes the process by which code can be contributed to DuraCloud
* [DuraCloud Release Process](release-new-version.md) - Describes the DuraCloud software release process
