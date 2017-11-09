# Initial DuraCloud Web Application Environment Setup

## Create Environment
1. Go to Beanstalk Service
2. Create application
3. Give it a name and description (e.g. DuraCloud)
4. Click create web server
5. Select "tomcat" platform and "load balancing" environment
6. Click through defaults until you reach the configuration details and select m3.large instance, your keypair, basic health reporting, and root volume device of 30 GiB with General Purpose SSD.
7. Select the IAM instance role you set up previously.
8. Launch the environment.

## Environment Variables
1. Navigate to environment -> configuration -> software configuration
    * jvm command line params:
      ""-Dduracloud.config.file=s3://<your-s3-config-bucket>/path-to-duracloud-properties-file"
    * environment params:
       * key: S3_CONFIG_BUCKET
       * value: <your-s3-config-bucket>

## Autoscaling
1. Set min/max instance counts to 5 and 10 respectively.
2. Under scaling trigger section:
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
1. Navigate to "Load Balancing"
2. Select session stickiness
3. Navigate to EC2 -> load balancers
4. Under port configuration enable "load balancer generated cookie stickiness" for ports 80 and 443.

You are now ready to deploy the DuraCloud beanstalk zip. You can do so by following the instruction in "Deploy to Production" detailed in [this document](release-new-version.md).

## DNS

In order to connect to DuraCloud accounts via the expected URL:

1. Register your preferred domain name via a domain registrar. This can be done using AWS Route 53.
2. Log in to the domain registrar, open the zone file editor for your DuraCloud domain
3. Add a CNAME record for each account subdomain which points to your DuraCloud environment URL. (The DuraCloud environment URL can be found on the Elastic Beanstalk dashboard for your the DuraCloud application.)