# AWS Setup

Before installing the components that together comprise DuraCloud, you'll need set up your aws environment.
Assuming you have an AWS account created, the following sections will guide you through this process.

## Setup IAM Roles / Users  
Create the following IAM Roles with the specified policies.

### duracloud-instance
Policies:  AmazonSQSFullAccess, AmazonSESFullAccess, AmazonS3ReadOnlyAccess
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
                "arn:aws:s3:::*.mill-bootstrap",
                "arn:aws:s3:::*.mill-bootstrap/*",
                "arn:aws:s3:::*.duplication-policy-repo",
                "arn:aws:s3:::*.duplication-policy-repo/*"
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

# Create Keypairs 
Navigate to EC2, select `Keypairs` and create new keypairs with the following names:
1. duracloud-keypair
1. management-console-keypair
1. mill-keypair

# Create and Configure VPC

1. Create a new VPC with name "duracloud-vpc"
    1. IPv4 CIDR Block:  10.0.0.0/16
    1. Select the newly created VPC and ensure that both "DNS Hostsnames" and "DNS Resolution" are both set to "Yes".
    This is important for EFS to work properly.
1. Create a subnet with name '10.0.1.0 <availability-zone> public'
    1. Select duracloud-vpc
    1. Specify the IPv4 CIDR Block: 10.0.1.0/24
    1. After saving,  select the subnet, click subnet actions, and select "Modify auto-assign IP settings" and check auto-assign box.
1. Optional: for higher availability and optimal spot pricing, create similar subnets in different availability zones.  For each new subnet, make
sure that you increment your IPv4 CIDR Blocks (ie 10.0.2.0/24, 10.0.3.0/24, 10.0.4.0/24, etc...)
1. Create an Internet Gateway named "duracloud-igw"
    1. Select the gateway and attach it to "duracloud-vpc"
1. Create a new route table named "duracloud-public-route" associated with your "duracloud-vpc"
    1. Select the new route, click routes tab, click "edit", and add a new route where destination is 0.0.0.0/0 and
    target is the duracloud-igw.
    

# Setup Security Groups
1. Navigate to EC2 -> Security Groups
1. create a new security group:  mill-sg
    1. Set the VPC to "duracloud-vpc"
    1. Add the following inbound rules: 
        1. SSH
        1. Use the "anywhere" source (ie. 0.0.0.0/0) or constrain to a limited set of IPs
1. create a new security group:  efs-sg
    1. Set the VPC to "duracloud-vpc"
    1. Add the following inbound rule: NFS using the mill-sg security group id into the NFS rule source field.
        

# Setup EFS
1. Navigate to EFS service and click create File System 
2. Give the EFS the name tag "duracloud-efs" and ensure that all available subnets are selected and that you are using 
the efs-sg security group.

# Setup SES
1. Go to the SES service in the AWS Console and create and verify a new email address.
1. [Read section 5 in this document in order to move your SES account out of the sandbox.](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/quick-start.html)

# SNS Topic
This step is very simple: Go to the SNS service in the AWS Console and create an SNS topic name "duracloud-account-topic".  This topic will be 
used by the management-console to notify topic subscribers of changes to the account database.

# DuraCloud configuration buckets
1. Create a bucket with the following name <domain>-production-config
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
   
   # can we get rid of these credentials? 
   notification.user=<aws access key>
   notification.pass=<aws password key>
   notification.from-address=<notification sender email address> 
   notification.admin-address=<duracloud admin email address>
   workDirectoryPath=/tmp/ama

    ```
    
### Set up your github key for accessing private repos
The cloud init scripts that provision the mill instances  depend on cloning 
the https://github.com/duraspace/puppet-duracloud-mill.git repository. This repository, however is currently private.
Until it becomes public,  we will need to grant you access to the repo.  Additionally you'll need to create a github ssh
keypair and drop the private key in the bucket you created in the last step.

     
###. Create CloudFront Signing Key and drop it in the config bucket
@TODO: Bill can you take me through this processes? 

