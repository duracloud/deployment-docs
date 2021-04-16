# Management Console Setup
## Prerequisites
Please make sure that you have followed all of the steps laid out in the [AWS Setup](aws-setup.md) document before
proceeding.

## Deploy to Beanstalk
### Create Beanstalk Environment
0. Go to the Beanstalk Service in the AWS console
0. Select `Create New Application`
0. Give it a name and description (e.g. DuraCloud Management Console)
0. Click `Create web server`
0. Select `Tomcat` platform, `Tomcat 8.5 with Corretto 11 running on 64bit Amazon Linux 2` platform branch and `4.1.7` version
0. Select the `Sample application` (it will be replaced by DuraCloud apps in a later step), and keep the default deployment preferences
0. Take defaults for environment name and URL (or update them if you'd prefer.) The environment URL must be unique.
0. Leave additional resources unchecked
0. Click on `Configure more options`
0. Under `Presets` click high availability
0. Edit `VPC` section and select your VPC and subnets and click save
0. Edit `Load Balancer` select application load balancer. Add a listener with https, port 443 and your *.<domain> certificate. Edit the default process and change the health check path to `/login`
0. Edit `Manage Updates` disable managed updates.
0. Click "Edit" in the `Software` section and select Apache under Container Options and enter the following Environmental Variables:
     * key: S3_CONFIG_BUCKET
          * value: ```<your-s3-config-bucket>```
          * key: AWS_REGION
          * value: ```<your-aws-region>``` ([make sure to use a valid EC2 region code](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions))
0. Edit `Capacity`
    0. select `Load balanced` Environment type 
    0. `min` instances to `2`
    0. `max` instances to`5`
    0. `m5.large` instance type 
    0. `scaling cooldown` to `360`.
    0. Scaling Triggers:
        * `Metric`: `CPUUtilization`
        * `Statistic`: `Average`
        * `Unit`: `Percent`
        * `Period`: `1`
        * `Breach Duration`: `5`
        * `Upper threshold`: `70`
        * `Scale up Increment`: `1`
        * `Lower threshold`: `20`
        * `Scale-down increment`: `-1`
0. Edit`Notifications`, enter an email address
0. Edit `Security`, set your keypair and IAM instance profile
0. Edit `Monitoring`
    * Enable `Ignore application 4xx`
    * Enable `Ignore load balancer 4xx`
0. Click `Create Environment`
0. Navigate to `Configuration -> Software` and set the followiwng:
    * jvm command line params:
      ```-Dduracloud.home=/tmp/duracloud-home -Dmc.config.file=s3://<your-s3-config-bucket>/path-to-duracloud-properties-file```

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
