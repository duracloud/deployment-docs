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
0. Select the IAM instance role you set up previously.
0. Launch the environment.

### Configure Environment Variables
0. Navigate to environment -> configuration -> software configuration
  * jvm command line params:
    ""-Dduracloud.config.file=s3://<your-s3-config-bucket>/path-to-duracloud-properties-file"
  * environment params:
     * key: S3_CONFIG_BUCKET
     * value: <your-s3-config-bucket>

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
0. Navigate to "Load Balancing"
0. Select session stickiness
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


