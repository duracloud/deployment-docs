# Initial DuraCloud Web Application Environment Setup

The DuraCloud web applications are run using AWS Elastic Beanstalk. This service manages the deployment of the applications, the configuration of load balancing, and the configuration of auto-scaling. The first step is to create an Elastic Beanstalk environment, followed by the configuration of each component.

## Create Environment

1. Go to the Beanstalk Service in the AWS console
2. Select `Create New Application`
3. Give it a name and description (e.g. DuraCloud)
4. Click `Create web server`
5. Select `Tomcat` platform and `Load balancing, auto scaling` environment
6. Select the `Sample application` (it will be replaced by DuraCloud apps in a later step), and keep the default deployment preferences
7. Take defaults for environment name and URL (or update them if you'd prefer.) The environment URL must be unique.
8. Leave additional resources unchecked
9. In configuration details, select an m3.large instance type, your keypair, 
   basic health reporting, and root volume device of 30 GiB with General Purpose SSD.
   1. Note: Enhanced health reporting in Beanstalk cannot be used with DuraCloud as it will report failures on HTTP 
   responses which have a 404 response code. This response code is perfectly valid for a REST API when an item that is requested does not exist. The DuraCloud SyncTool makes frequent use of requests to check for the existence of files prior to uploads, which often result in 404 responses. Using Enhanced health reporting with Beanstalk will result in functional DuraCloud instances being taken out of service.
7. Select the IAM instance role you set up previously for the duracloud instance (`uracloud-instance`).
8. Launch the environment.

## Environment Variables
1. Navigate to environment -> configuration -> software configuration
    * jvm command line params:
      ```-Dduracloud.config.file=s3://<your-s3-config-bucket>/path-to-duracloud-properties-file```
    * environment params:
       * key: S3_CONFIG_BUCKET
       * value: ```<your-s3-config-bucket>```

## Autoscaling
1. Set min/max instance counts based on your system needs.
   1. An initial minimum of 3 and maximum of 7 should be sufficient. When consistent load is higher you may choose to increase these numbers.
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

## SSL

In order to configure SSL, you must first have a valid SSL certificate for your domain.

* It is recommended that a wildcard SSL certificate be used, as that will allow all subdomains to be covered.
* The SSL certificate can be created through Route 53, if Route 53 is your domain registrar (or if you've transferred control of your domain to Route 53.) If not using Route 53, you will need to purchase an SSL certificate from a certificate authority. SSL certificates are often available from domain registrars.
* If you are using Route 53 to create an SSL certificate, it is automatically included in IAM for use in Elastic Beanstalk. If you have used a different account (as for instance the root account) to register the domain, you need to create the certificate from the AWS Account used to run the DuraCloud infrastructure. If not using Route 53, [you will need to import your certificate to IAM](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/configuring-https-ssl-upload.html).
* Once the certificate is in place in IAM, go back to Elastic Beanstalk -> Configuration -> Load Balancing
* In the dropdown next to `SSL certificate ID` select your certificate
* Select `Apply`

## DNS

In order to connect to DuraCloud accounts via the expected URL:

1. Register your preferred domain name via a domain registrar. This can be done using AWS Route 53.
2. Log in to the domain registrar, open the zone file editor for your DuraCloud domain
3. If you are using Route53 add an A ALIAS record, you can also use * here to map all future subdomains to the EB DuraCloud-Env. If you use an external DNS registrar, add a CNAME record for each account subdomain which points to your DuraCloud environment URL. (The DuraCloud environment URL can be found on the Elastic Beanstalk dashboard for your the DuraCloud application.)
