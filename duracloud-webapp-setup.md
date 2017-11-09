# Initial DuraCloud Web Application Environment Setup

## Create Environment
0. Go to Beanstalk Service
1. Create application
2. Give it a name and description (e.g. DuraCloud)
3. Click create web server
4. Select "tomcat" platform and "load balancing" environment
5. Click through defaults until you reach the configuration details and select m3.large instance, your keypair, basic health reporting, and root volume device of 30 GiB with General Purpose SSD.
6. Select the IAM instance role you set up previously.
7. Launch the environment.

## Environment Variables
0. Navigate to environment -> configuration -> software configuration
   * jvm command line params:
     ""-Dduracloud.config.file=s3://<your-s3-config-bucket>/path-to-duracloud-properties-file"
   * environment params:
      * key: S3_CONFIG_BUCKET
      * value: <your-s3-config-bucket>

## Autoscaling
0. Set min/max instance counts to 5 and 10 respectively.
1. Under scaling trigger section:
  * Trigger measurement: CPU Utilization
  * Trigger statistic: average
  * Unit of measurement: percent
  * Measurement period: 1
  * Breach duration: 1
  * Upper threshold: 80
  * Upper breach scale: 1
  * Lower threshold: 20
  * Lower breach scale: -1

## Load Balancer
0. Navigate to "Load Balancing"
1. Select session stickiness
2. Navigate to EC2 -> load balancers
3. Under port configuration enable "load balancer generated cookie stickiness" for ports 80 and 443.

You are now ready to deploy the DuraCloud beanstalk zip. You can do so by following the instruction in "Deploy to Production" detailed in [this document](release-new-version.md).
