# Creating New Accounts

### Creating accounts in the DuraCloud system requires:

1. Getting details from the customer
2. Creating a new AWS account
3. Creating a new DuraCloud account via the Management Console
4. Configuring DNS
5. Providing access to the DuraCloud account
6. Adding a Duplication Policy (if necessary)

### Customer details

Before starting down the path of creating a new DuraCloud account, you will need to gather the following information from the customer:

1. **The customer name.** This will be the account name in DuraCloud. This is often the name of an institution, organization, or department.
2. **The subdomain for their DuraCloud account.** This will define the URL through which they access their DuraCloud account. This name should be relatively short (generally less than 20 characters) and include only lowercase letters, numbers, and dashes. Ensure that the subdomain selected is not already in use by another account. The primary domain name of the institution is a popular subdomain choice. 
3. **Required storage providers and purchased storage TBs.** The type of DuraCloud account will determine if there is a need to create more than the primary AWS account. The expected storage total is captured and compared to actual values to provide information about storage percentage consumed.
4. **Email addresses of DuraCloud users.** This does not need to be a comprehensive list (as users can be added easily), but account setup cannot be considered complete until the customer has at least one user with access to their account. Depending on the type of account (Enterprise or Preservation), one or more users may need to be given Administrator privileges.
5. **Names of spaces to create.** For non-Enterprise accounts, spaces will need to be created and access given to account users. The number of spaces is generally limited (to < 10) to keep this manageable, but exceptions can be made.

### Create a new AWS account

DuraCloud requires an independent AWS account for each customer storage provider. This approach allows storage utilization to easily be determined per account and provides a simple means of division between accounts. In order to support the growth of AWS accounts, it is recommended that one AWS account be designated as the master account through the [configuration of AWS Organizations](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_create.html), which was previously known as Consolidated Billing in AWS. 

Once AWS Organizations has been configured, you will start the process of creating a new account on the [Organizations home page](https://console.aws.amazon.com/organizations/home). 

1. Create the AWS account

   1. On the Accounts tab, select the `Add account` button, followed by `Create account`
   2. In the `Full name` field, enter the name of the customer account (likely the customer name). You may choose to prefix this name with `DuraCloud: ` to make clear that this is a DuraCloud sub-account.
   3. In the `Email` field enter an email address to be associated with this account. This address needs to be unique for this AWS account. (If your master account uses a Gmail-based email address, you can use the `+` sign to [create addresses which are unique](https://gmail.googleblog.com/2008/03/2-hidden-ways-to-get-more-from-your.html) for this purpose.)
   4. Leave the `IAM role name` field blank
   5. Select `Create`

2. Retrieve AWS account credentials

   1. Log out of the AWS console (or, better yet, open a different browser) and go to: <https://console.aws.amazon.com/>
   2. Enter the email address for the account you just created, select `Next`
   3. Select `Forgot Password`, enter the Captcha, select `Submit`. This will result in an email being sent to the associated email account.
   4. Look for an email with subject `Amazon Web Services Password Assistance`; inside will be a link. Click the link and enter a new password.
   5. Sign in to the AWS console with your new account credentials.
   6. Store the credentials and new account details in a secure location
   7. Further [details about this process can be found in the AWS documentation](http://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_accounts_access.html#orgs_manage_accounts_access-as-root)

3. Create an IAM user for use with DuraCloud

   1. In the AWS console, select `Services` and then `IAM` to enter the IAM console
   2. Select `Users` and `Add User`
   3. For `User name` enter `duracloud-user`, then check the box for `Programmatic access` and select `Next:Permissions`
   4. Select `Next: Review` (we will set permissions in a moment) and then `Create user`
   5. Capture the user's Access Key ID and Secret Key, then `Close`
   6. Select `duracloud-user`, then on the `Permissions` tab, select `Add inline policy`
   7. Add the following two policies, each as a Custom Policy:
      `storage-provider-access-policy` 
      ```json
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Sid": "Stmt1436886535000",
            "Effect": "Allow",
            "Action": [
              "s3:*",
              "cloudfront:*"
            ],
            "Resource": [
              "*"
            ]
          }
        ]
      }
      ```
      `cloudwatch-policy-for-storage-reports`
      ```json
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Sid": "Stmt1462574852000",
                  "Effect": "Allow",
                  "Action": [
                      "cloudwatch:GetMetricData",
                      "cloudwatch:GetMetricStatistics",
                      "cloudwatch:ListMetrics"
                  ],
                  "Resource": [
                      "*"
                  ]
              }
          ]
      }
      ```
If more than one AWS account is required (such as for a secondary Glacier account), then this process will need to be followed twice.

### Create a new DuraCloud account

1. Log into the DuraCloud Management Console as a user with root level access
2. Select the `Root Console` link in the upper right-hand corner of the UI
3. Select `Add Account` on the Accounts tab
4. Enter the account details and select `Next >>`
5. Select the storage provider(s) which will be associated with the account, and select the radio button next to the primary provider (AMAZON_S3 will usually be the primary), then select `Finish`. At this point the DuraCloud account exists, but the providers still need to be configured.
6. Back on the Accounts tab, find the account which you just added and select `Configure Providers`
7. For each provider, enter the Username and Password values. These are the Access Key ID (username) and Secret Access Key (password) values from the duracloud-user you created in IAM in the last section. **Note: that these are not the email address and password for the AWS account.** 
8. Update the storage limit for each provider (these are generally the same number), then select `Activate`. As you might expect, this will activate the account in the DuraCloud system.

### Configure DNS

In order to be able to connect to the new DuraCloud account with the expected subdomain, the subdomain will need to be added to DNS.

1. Log in to your DNS provider (where the DNS configuration for your DuraCloud system is managed) 
2. Open the zone file editor for your DuraCloud domain
3. Add a CNAME record to point the new account subdomain to your DuraCloud environment URL. (The DuraCloud environment URL can be found on the Elastic Beanstalk dashboard for your the DuraCloud application.)
4. After a few minutes (it may take a little while for DNS to resolve), you should be able to log in as a root user to the newly created DuraCloud account.

### Provide access to the DuraCloud account

In order to be able to access DuraCloud, users will need to first create an account profile. This is done by selecting the `Create New Profile` link under the login prompt for the Management Console. You can capture this link and send it to any user who needs to create an account.

Once users have been added to the DuraCloud system, you will need to give them access to their account.

1. Find the account on the Management Console home page
2. Select `Manage Users and Roles`
3. From here you can add their username to the `Username` field and select `Add`. (Note that there is also an option to invite users via email. While this is functional, some users find the process confusing.)
4. Once a user is added, you can update their permissions to be an Administrator (if this is an Enterprise account, and they are the selected Admin.) Once an Administrator, they have full access to add, update, and remove users in their account.

For non-Enterprise accounts it is often convenient to create a group which includes all users in the account. This group can be used to easily provide access to all or some spaces to everyone in the account.

1. Find the account on the Management Console home page
2. Select `Manage Groups`
3. Enter a group name and select `Add New Group`
4. Select all users and select `Add`
5. Select `Save`

For non-Enterprise accounts, you will need to create spaces and provide access to these spaces for your users.

1. Log in to the new DuraCloud account
2. Create the list of spaces requested by the customer
3. For each space, under `Permissions`, select `Add`, then check the boxes next to the necessary access levels and select `Save`

### Add a Duplication Policy

If the new DuraCloud account has multiple storage providers, you will need to add a duplication policy

1. Open the Duplication Policy Editor
2. Enter your DuraCloud credentials and the subdomain of the DuraCloud account where the duplication policies are hosted, then select `Sign In`
3. Enter the subdomain of your new account in the top box, this will create an empty duplication policy
4. For any spaces which exist, use the editor to add an entry to indicate the source and destination storage providers (often S3 and Glacier, respectively.)
5. Note that you will need to update the Duplication Policy as spaces are added to the account. [There is current development activity around removing this requirement.](https://jira.duraspace.org/browse/DURACLOUD-979)

### 





