# AWS Setup

Before installing the components that together comprise DuraCloud, you will need set up your AWS environment.

## Create an AWS account

You will need an AWS account to provision AWS services for the deployment of DuraCloud. In order to support the growth of AWS accounts as your DuraCloud client base grows, it is recommended that one AWS account be designated as the master account through the [configuration of AWS Organizations](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_create.html), which was previously known as Consolidated Billing. 

A sub-account should then be created which will be used for compute-related AWS resources. Once your sub-account is created, [log in to it](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html#orgs_manage_accounts_access-as-root) and create an IAM user with Administrative authority. Log in as this IAM user to proceed with the setup detailed below.

## Set up IAM Roles / Users  
Create the following IAM Roles with the specified policies. These IAM roles are used to provide the DuraCloud applications and systems with the necessary access to AWS services.

### duracloud-instance
Policies:  AmazonSNSFullAccess, AmazonSQSFullAccess, AmazonSESFullAccess, AmazonS3ReadOnlyAccess
### management-console  
Policies:  AmazonSESFullAccess, AmazonSNSFullAccess,AmazonS3FullAccess
### duracloud-mill  
Attach the following inline policies:  

**cloudwatch-policy**
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1423955197000",
            "Effect": "Allow",
            "Action": [
                "cloudwatch:PutMetricData"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
**s3-policy**
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1389129594000",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Sid": "Stmt1423673190000",
            "Effect": "Allow",
            "Action": [
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::*.auditlogs",
                "arn:aws:s3:::*.auditlogs/*"
            ]
        },
        {
            "Sid": "Stmt1389129682000",
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
**ses-policy**
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1389218374000",
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
**sqs-policy**
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1389129844000",
            "Effect": "Allow",
            "Action": [
                "sqs:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

## Create Keypairs 
Navigate to EC2, select `Keypairs` and create new keypairs with the following names. These keypairs are associated with EC2 instances as they are started. The keypairs allow SSH access to these instances.
1. duracloud-keypair
2. management-console-keypair
3. mill-keypair

## Create and Configure VPC
VPC is used for the deployment of DuraCloud Mill instances.

1. Create a new VPC with name "duracloud"
    1. IPv4 CIDR Block:  10.0.0.0/16
    2. Select the newly created VPC and ensure that both "DNS Hostsnames" and "DNS Resolution" are both set to "Yes".
      This is important for EFS to work properly.
2. Create a subnet with name '10.0.1.0 `availability-zone` public'
    1. Select duracloud
    2. Specify the IPv4 CIDR Block: 10.0.1.0/24
    3. After saving,  select the subnet, click subnet actions, and select "Modify auto-assign IP settings" and check auto-assign box.
3. Optional: for higher availability and optimal spot pricing, create similar subnets in different availability zones.  For each new subnet, make
  sure that you increment your IPv4 CIDR Blocks (ie 10.0.2.0/24, 10.0.3.0/24, 10.0.4.0/24, etc...)
4. Create an Internet Gateway named "duracloud-igw"
    1. Select the gateway and attach it to "duracloud"
5. Create a new route table named "duracloud-public-route" associated with your "duracloud"
    1. Select the new route, click routes tab, click "edit", and add a new route where destination is 0.0.0.0/0 and
      target is the duracloud-igw.

## Set up Security Groups
1. Navigate to EC2 -> Security Groups in the AWS console
2. Create a new security group:  mill-vpc
    1. Set the VPC to "duracloud"
    2. Add the following inbound rules: 
        1. SSH
        2. Use the "anywhere" source (ie. 0.0.0.0/0) or constrain to a limited set of IPs
3. Create a new security group:  efs-vpc
    1. Set the VPC to "duracloud"
    2. Add the following inbound rule: NFS using the mill-vpc security group id into the NFS rule source field.

## Set up EFS
1. Navigate to EFS in the AWS console
2. Click `Create File System `
3. Give the EFS the name tag "duracloud-efs" and ensure that all available subnets are selected and that you are using the efs-sg security group.

## Set up SES
1. Go to the SES service in the AWS Console
2. Create and verify a new email address.
3. [Read section 5 in this document in order to move your SES account out of the sandbox.](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/quick-start.html)

## Set up SNS Topic
1. Go to the SNS service in the AWS Console
2. Create an SNS topic name "duracloud-account-topic".  This topic will be used by the management-console to notify topic subscribers of changes to the account database. Subscribers for this topic will be the DuraCloud applications on each instance.
3. Create at least one SNS topic to allow for email notifications from Elastic Beanstalk and Amazon Cloudwatch. You may want distinct topics for DuraCloud and Mill notifications. Assign at least one email subscriber to each topic (an address where you would like to receive DuraCloud notifications) and verify the subscription via the email link that is sent by SNS.

## Set up S3

### DuraCloud configuration buckets

1. Create a bucket for production DuraCloud configuration (`domain`-production-config)
2. Add the following file "duracloud-config.properties" to it: 
   ```
   mill.db.name=duracloud_mill
   mill.db.host=<your-database-host>
   mill.db.port=3306
   mill.db.user=millreader
   mill.db.pass=<mill reader password>
    
   db.name=duracloud_accounts
   db.host=<your-database-host>
   db.port=3306
   db.user=accountadmin
   db.pass=<account admin password>

   mc.host=<your management console host>
   mc.port=443
   mc.context=
   mc.domain=<your management console domain>
   
   # Recaptcha
   recaptcha.enabled=true
   recaptcha.siteKey=<your recaptcha site key>
   recaptcha.secret=<your recaptcha secret>

   # DuraCloud versions >= 4.4.3 allow for leaving out the notification credentials
   notification.user=<aws access key>
   notification.pass=<aws password key>
   notification.from-address=<notification sender email address> 
   notification.admin-address=<duracloud admin email address>
   workDirectoryPath=/tmp/ama

   ```
3. Even if you're not planning to use sumologic for log aggregation and visualization you
   must place the a sumo.conf file in this bucket.  You can use dummy values for the credentials.
   At some point we may want to handle this more elegantly.
   ```
   accessid=<sumo-access-id>
   accesskey=<sumo-access-key>
   rpmAutoStart=false
   syncSources=/etc/sumo_sources.json

   ```

### Set up your github key for accessing private repos
The cloud init scripts that provision the mill instances  depend on cloning 
the https://github.com/duraspace/puppet-duracloud-mill.git repository. This repository is private, so we will need to grant you access to the repo.  Additionally you'll need to create a github ssh
keypair and drop the private key in the bucket you created in the last step.

### Create CloudFront Signing Key

A CloudFront Signing Key is used by DuraCloud to digitally sign requests for streaming content. These signatures are required to support spaces for which secure streaming is enabled. An authenticated request for access to a secure stream via the DuraCloud API results in a signed RTMP streaming URL. This URL can then be used to stream the media file within the constraints defined by the original request.

1. Log in to the AWS console and navigate to the [IAM Security Credentials page](https://console.aws.amazon.com/iam/home?#/security_credential)
2. Open the CloudFront key pairs section 
3. Create a new key pair, making sure to capture the **private key**, and **Access Key ID**
4. Ensure that the new key pair is active
5. Reformat the private key for use in Java (from PEM to DER format)
   `openssl pkcs8 -topk8 -nocrypt -in origin.pem -inform PEM -out new.der -outform DER`
6. Add the new DER formatted file to the production-config S3 bucket created above and capture the path of the file in S3, it will be something like:
   `s3://<domain>-production-config/production-cloudfront-signing-key.der`
7. In the AWS console, navigate to [Account Settings](https://console.aws.amazon.com/billing/home?#/account) and capture the Account ID
8. When [configuring the Management Console](management-console-setup.md) you will use the Account ID, Access Key ID, and S3 path to configure `Global Properties`

### Create shared buckets
Due to a long ago adopted DuraCloud convention related to preventing all buckets from showing up in a DuraCloud space listing as well as guarantee uniqueness of spaces within a single account,  there are two buckets that you'll need to create with a quirky prefix.  That prefix must be 20 lowercase alpha-numeric characters followed by a period. For example: akiajammmxxxplax656a."  Look familiar?  Yes, by default this is the AWS Access Key ID. Any string of alphanumeric characters will do as long as they are unique.  So create two buckets:   
* yourtwentycharprefix.auditlogs
* yourtwentycharprefix.duplication-policy-repo

In the duplication policy repo you must drop a file named "duplication-accounts.json" that contains a single empty json array - namely:
```
[]
```
Further content and configuration of the duplication-policy-repo bucket is included in the [Mill configuration](mill-setup.md).
