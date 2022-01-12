# Initial DuraCloud Web Application Environment Setup

The DuraCloud web applications are run using AWS Elastic Beanstalk. This service manages the deployment of the applications, the configuration of load balancing, and the configuration of auto-scaling. The first step is to create an Elastic Beanstalk environment, followed by the configuration of each component.

## Create Environment

0. Go to the Beanstalk Service in the AWS console
0. Select `Create New Application`
0. Give it a name and description (e.g. DuraCloud)
0. Click `Create web server`
0. Select `Tomcat` platform, `Tomcat 8.5 with Corretto 11 running on 64bit Amazon Linux 2` platform branch and `4.2.10` version
0. Select the `Sample application` (it will be replaced by DuraCloud apps in a later step), and keep the default deployment preferences
0. Take defaults for environment name and URL (or update them if you'd prefer.) The environment URL must be unique.
0. Leave additional resources unchecked
0. Click on `Configure more options`
0. Under `Presets` click high availability
0. Edit `VPC` section and select your VPC and subnets and click save
0. Edit `Load Balancer` select application load balancer. Add a listener with https, port 443 and your *.<domain> certificate. Edit the default process and change the health check path to /duradmin/login
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
      ```-Dduracloud.config.file=s3://<your-s3-config-bucket>/path-to-duracloud-properties-file```

## Load Balancer Stickiness config (EC2)

There are a couple of additional configurations you'll need to make in EC2 -> Load Balancer section in order to support sticky sessions with an Application Load Balancer.

0. Click on your DuraCloud autoscaling group
0. Scroll down to `Load Balancing` and click on the target group.
0. Click on the `Attributes` tab and hit `Edit`
0. Select `Application-based cookie` under `Stickiness type`
0. Set the `App cookie name` to `JSESSIONID`
0. Save changes.

You are now ready to deploy the DuraCloud beanstalk zip. You can do so by following the instruction in "Deploy to Production" detailed in [this document](release-new-version.md).

## SSL

In order to configure SSL, you must first have a valid SSL certificate for your domain.

* It is recommended that a wildcard SSL certificate be used, as that will allow all subdomains to be covered.
* The SSL certificate can be created through Route 53, if Route 53 is your domain registrar (or if you've transferred control of your domain to Route 53.) If not using Route 53, you will need to purchase an SSL certificate from a certificate authority. SSL certificates are often available from domain registrars.
* If you are using Route 53 to create an SSL certificate, it is automatically included in IAM for use in Elastic Beanstalk. If you have used a different AWS account (such as another account in your AWS Organization) to register the domain, you will need to create the certificate from the AWS account used to run the DuraCloud infrastructure in order for it to be available in Beanstalk.
* If not using Route 53, [you will need to import your certificate using thw AWS Certificate Manager (ACM)](https://docs.aws.amazon.com/acm/latest/userguide/import-certificate-api-cli.html).
* Once the certificate is in place in ACM, go back to Elastic Beanstalk -> Configuration -> Load Balancer -> Edit
* Select the 443 port and Edit, then in the dropdown below `SSL certificate` select your certificate
* Select `Save`, then `Apply`

## DNS

In order to connect to DuraCloud accounts via the expected URL:

1. Register your preferred domain name via a domain registrar. This can be done using AWS Route 53.
2. Log in to the domain registrar, open the zone file editor for your DuraCloud domain
3. If you are using Route53 add an A ALIAS record, you can also use * here to map all future subdomains to the EB DuraCloud-Env. If you use an external DNS registrar, add a CNAME record for each account subdomain which points to your DuraCloud environment URL. (The DuraCloud environment URL can be found on the Elastic Beanstalk dashboard for your the DuraCloud application.)
