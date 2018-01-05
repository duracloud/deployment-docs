# Management Console Setup
## Prerequisites
Please make sure that you have followed all of the steps laid out in the [AWS Setup](aws-setup.md) document before
proceeding.

## Deploy to Beanstalk
### Create Beanstalk Environment
0. Go to Beanstalk Service
0. Create application
0. Give it a name and description (e.g. Management Console)
0. Click create web server
0. Select "tomcat" platform and "load balancing" environment
0. Click through defaults until you reach the configuration details and select m3.large instance, your keypair, basic health reporting, and root volume device of 30 GiB with General Purpose SSD.
0. Select the IAM instance role you set up previously (ie `management-console`).
0. Launch the environment.

### Configure Environment Variables
0. Navigate to environment -> configuration -> software configuration
  * jvm command line params:
    ```-Dduracloud.home=/tmp/duracloud-home -Dmc.config.file=s3://<your-s3-config-bucket>/path-to-duracloud-properties-file```
  * environment params:
     * key: S3_CONFIG_BUCKET
     * value: ```<your-s3-config-bucket>```

### Configure Autoscaling
0. Set min/max instance counts to 2 and 3 respectively.
0. Under scaling trigger section:
  * Trigger measurement: CPU Utilization
  * Trigger statistic: average
  * Unit of measurement: percent
  * Measurement period: 1
  * Breach duration: 1
  * Upper threshold: 80
  * Upper breach scale: 1
  * Lower threshold: 20
  * Lower breach scale: -1

### Configure Load Balancer
0. Secure listener port: 443
0. Navigate to "Load Balancing"
0. Select SSL Certificate
0. Select session stickiness
0. Click `Apply`
0. Navigate to EC2 -> load balancers
0. Under port configuration enable "load balancer generated cookie stickiness" for ports 80 and 443.

You are now ready to deploy the DuraCloud beanstalk zip. You can do so by following the instruction in "Deploy to Production" detailed in [this document](release-new-version.md).

1. Build the latest tagged release of the management-console 
    ```
    git clone https://github.com/duracloud/management-console.git
    cd management-console
    mvn clean install -DskipTests -DskipIntTests -DskipDeploy
    ```
1. Upload the account-management-app/target/ama-<version>.war to Beanstalk (Application Versions).

## Configure Management Console

### Create a root user
0. Create a user by clicking on the new user link on the management console login page.
0. Make the newly created user a root user by logging directly into the duracloud_accounts database and runing 
    the following command: 
    ```update duracloud_user set root = true;```
    
### Configure Mill properties
0. Login into the management console.
0. Click on `Root Console` in the upper right hand corner of window.
0. Click on `DuraCloud Mill` tab.
0. Enter the requested mill database fields.
0. Enter "auditlogs" for the Audit Log Space Id
0. In a separate window,  log into the aws console and navigate to SQS.  Note the name of the 
   queue ending in "_audit" and enter that into the DuraCloud Mill form you were just working on.
0. Click `Ok`

### Configure Global Properties
0. Click on the `Global Properties` tab.
0. In a separate tab go into the AWS SNS console, copy the topic ARN for the `duracloud-account-topic` for the topic your created in the SNS step in the [AWS Setup](aws-setup.md) 
   document, and paste into the `Instance Notification Topic ARN` field.
0. Retrieve your CloudFront account id, the access key id and s3 path to your CloudFront key.
   These values you set aside in the CloudFront Key generation step in the [AWS setup](aws-setup.md)
   document and plug them into their respecitive fields in the form. 
0. Now you are ready to [start creating accounts](creating-new-accounts.md). The first account you create should be the account that will be used to store and access your auditlogs and duplication-policy-repo, as noted in the instructions for the (duplication-policy-editor](mill-setup.md#deploy-the-duplication-policy-editor)
