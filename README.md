# DuraCloud Deployment Documentation

The following documents capture the steps required to build, release, and deploy DuraCloud software

A special thanks to the [DuraCloud Europe](https://www.duracloudeurope.org/) team at [4Science](https://www.4science.it) for contributing to and funding the development of these documents! :thumbsup:

### Initial setup
* [General AWS Environment Setup](aws-setup.md) - Describes the initial setup of an AWS account and the services which support DuraCloud deployment
* [Database setup](database-setup.md) - Describes the setup of Amazon RDS to support DuraCloud
* [Mill deployment](mill-setup.md) - Defines the steps required to deploy and configure the DuraCloud Mill
* [DuraCloud Application](duracloud-webapp-setup.md) - Describes the process of deploying the DuraCloud applications (DuraStore and DurAdmin) using AWS Elastic Beanstalk
* [Management Console](management-console-setup.md) -  Describes the process of deploying the DuraCloud Management Console using AWS Elastic Beanstalk
* [Log aggregation](logging.md) - Describes log capture and visualization using SumoLogic
* [DuraCloud Bridge](https://wiki.lyrasis.org/display/DURACLOUDDOC/DuraCloud+Bridge) - (External link) Describes the purpose, API, and configuration of the DuraCloud Bridge application

### Account Management
* [Creating new accounts](creating-new-accounts.md) - Describes how a new client DuraCloud account is added to the system
* [Closing accounts](closing-accounts.md) - Describes how to deactivate, delete, and transition DuraCloud accounts

### Architecture
* [DuraCloud Architecture](architecture.md) - Describes the overall DuraCloud architecture and how the DuraCloud components operate together

### Software Development
* [Development Guide: Getting Started](dev-guide-getting-started.md) - Learn how to set up your development environment to work on DuraCloud software
* [DuraCloud code contribution guidelines](https://github.com/duracloud/duracloud/blob/master/CONTRIBUTING.md) - (External link) Describes how to get started contributing to the DuraCloud project (coding not required!)
* [DuraCloud Release Process](release-new-version.md) - Describes the DuraCloud software release process
