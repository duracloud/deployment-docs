# Closing Accounts

For a variety of reasons, accounts must occasionally be closed and removed from DuraCloud. Most of this work can be done directly from the Management Console, but in the case where storage accounts are to be transitioned to the data owner, additional steps are required.

## Closing an account - removing content

To close out an account in DuraCloud where content will be removed you will need to:

1. Coordinate with the customer to ensure that they retrieve all of their content. (If the customer would prefer to take ownership of the storage accounts, see below.)
2. Remove all content and spaces. This can be done using the DuraCloud UI by selecting all spaces and choosing `Delete`. Note that the spaces will delete from the UI initially, but a refresh of the Spaces tab will show they still exist. In the background the content is being deleted, and when all content is gone, the space will be deleted as well. This process can take some time, especially if there is significant content in the spaces. You will want to keep an eye on this process to ensure the deletes continue. If the deletes stop for a space, simply delete it again to get them going.
3. Once all content is removed, go the Management Console, find the account, and select to Deactivate it. Deactivating an account retains all account settings, but the account is no longer available through DuraCloud. You may choose to leave the account in the deactivated state for as long as needed, or reactivate it at any time.. Virtually no resources are consumed by accounts in the deactivated state.
4. If you'd like to remove the account completely, go the Root Console, find the account, and select Delete. This will remove the account along with all of its settings and storage providers. This will not remove users associated with the account, but it will remove the association between those users and the account.
5. If the account has a duplication policy (i.e. it is replicating content to secondary storage), remove the duplication policy for the account via the policy editor. 
   1. Open the Duplication Policy Editor.
   2. Enter your DuraCloud credentials and the subdomain of the DuraCloud account where the duplication policies are hosted, then select      `Sign In`. You will see the list of all DuraCloud accounts for which duplication is configured.
   3. Click on the trash icon to delete the duplication policy for the subdomain of the account you have closed.
6. Remove the subdomain CNAME from the zone file of your domain registrar (e.g. GoDaddy).

## Closing an account - transitioning AWS account

If the customer would prefer to maintain content stored in the underlying storage providers (e.g. Amazon S3), follow this process to close the DuraCloud account and transition ownership of stored content:

1. Coordinate with the customer to determine the point of contact (POC) for the transition. This person must have access to the customer's AWS Organization account and be authorized to take possession of AWS accounts. 
2. Coordinate with the customer to ensure that they have stopped all content transfers (and verify this is the case)
3. If there are multiple stores to be transitioned, run a Mill Duplication process to ensure all file duplication is up-to-date
4. (Optional) Export the manifest and audit logs for each existing space to an external file
5. (Optional) Create a space for manifest and audit log files, store exported files in this new space
6. Delete the DuraCloud account via the Management Console (see above for more details)
7. In the AWS Console:
   1. Remove any existing root-level Access Keys (found under Security Credentials)
   2. Create an IAM user with admin rights for the customer POC, and have them log in
   3. In Payment methods, have the customer POC enter a card, then remove any existing (DuraSpace) cards
   4. In AWS Organizations, leave the master DuraSpace organization (and verify the disconnect from the DuraSpace org)
   5. In AWS Organizations, add the account to the customer organization (and have the customer POC verify the connection)
   6. Remove the DuraCloud IAM user
   7. Have the customer POC change the Account Contact Information
   8. Change root Account Name, email address, and password to values defined by the customer
   9. Log out of the account. Have the customer POC change the root password.
8. If there are multiple stores to be transferred, complete these steps for each account.
