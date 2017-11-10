# DuraCloud Architecture

A high-level view of DuraCloud's components can be found as part of the [architecture diagrams included in the user documentation](https://wiki.duraspace.org/display/DURACLOUD/DuraCloud+Architecture). The purpose of this document is to describe how those components operate together from a functional standpoint.

The DuraCloud application components are made up of 3 primary Java web applications: DuraStore, DurAdmin, and the Management Console. DuraStore is the core of DuraCloud, it manages the flow of content through its REST API to the cloud-based storage systems. DurAdmin provides a graphical interface to allow interacting with DuraStore. Both DuraStore and DurAdmin provide a view into an individual DuraCloud account. Those accounts are managed in the Management Console.

## DuraCloud Accounts

DuraCloud accounts are managed by the Management Console and stored in the Account database. This database includes the following information:

* Customer details: Account name, organization name, department, account subdomain
* Storage details: Storage provider accounts type, credentials, and storage limit
* User details: User names, hashed passwords, email addresses
* Group details: Group name, users included in the group
* Mill details: Communication details for the Mill
* Rights details: User access levels and account associations
* System properties: Details needed for system operations

The Management Console is used to add, edit, and remove DuraCloud accounts as well as add, edit, and remove DuraCloud users. All changes are stored in the Account database.

The DuraCloud applications (DuraStore and DurAdmin) retrieve and cache a copy of the details from the Account database. This allows each of the applications in a DuraCloud instance to validate user credentials and connect to the correct storage providers for any account requested. This is an important component of the multi-tenancy capability of DuraCloud. Any DuraCloud instance is thus able to service any account, so all requests can be load balanced across all available instances.

In order to accommodate changes, the Management Console publishes all updates to an SNS topic. On startup, each DuraCloud instance subscribes to this topic. When a change event occurs, the DuraCloud instances refresh their cache.

## Spaces

All content stored in DuraCloud is stored in a **Space**. The term "space" was selected as an alternative to "bucket" (as used by Amazon S3) or "container" (as used by OpenStack). 

​	*A bit of DuraCloud trivia: The service now known as DuraCloud was originally code-named DuraSpace before the organization known as DuraSpace was created. This was part of the reasoning behind the use of "space" as the alternative to "bucket" and "container".*

Not only are files grouped by Spaces, but many functions in DuraCloud occur at the Space level. Examples of this are access controls (which are designed to allow read and/or write access to all content store in a space) and media streaming (which allows streaming for all content within an enabled space.)

## File Management

DuraCloud manages the storage and preservation of files. All files which are stored in DuraCloud pass, one way or another, through the [DuraStore REST API](https://wiki.duraspace.org/display/DURACLOUDDOC/DuraCloud+REST+API). No content is stored in DuraCloud without first passing through this gateway. The reason for this is to ensure that DuraCloud can keep track of all stored content. That tracking system is called the **Space Manifest**

When a PUT call is used to add a file to DuraCloud (both the DurAdmin UI and the SyncTool use the same PUT call to store content), part of the processing of that file includes adding a message to an SQS **Audit Queue**. 

Like all queues in the DuraCloud system, the Audit Queue is processed by the DuraCloud Mill. The purpose of the Mill is to handle all the background processing for DuraCloud. DuraCloud uses a variety of SQS queues to handle this processing. Any time work needs to be done, a message is added to one of these queues. These messages are then retrieved by one of the Mill worker instances. A process called *workman* (for Work Manager) handles pulling down work from a queue for a worker instance and distributing it to one of the worker threads on that instance.

An **Audit Task** is one of the types of work that the Mill worker can process. The Audit Queue only captures Audit Tasks. Other queues capture other task types, such as Duplication or Bit Integrity. The processing of an Audit Task performs two primary actions:

1. The change is captured in the **Audit Log**
2. The change is captured in the **Space Manifest**

The Audit Log is a running list of all actions which occur that change the state of a space. These actions include adding a file, updating a file, changing the properties of a file, and deleting a file.

The Space Manifest is a record of the current state of a space; it lists all files which are currently contained in a space, along with the checksum of that file.

To know the history of a space, the Audit Log is consulted. To know the current state of a space, the Space Manifest is consulted. Both of these lists are captured in the Mill database. The manifest remains in the database (in the manifest_item table) to enable simple querying. The Audit Log entries are contained in the database (in the audit_log_item table) for a short time period before being written to an external file in S3.

Both the Manifest and Audit Log for a space are available to users and administrators through the DuraCloud REST API and UI.

The other primary action which occurs in a PUT request is that the file in the body of the request is stored. Where the file is stored depends on the storage providers associated with the account.

Of course, other actions also occur to files, they can be updated or deleted, or their properties can be 

## Mill Processing

As noted above, the DuraCloud Mill is used to handle the background processing for the DuraCloud system. The primary activities of the Mill are to perform the following actions:

* Capture audit events in the Audit Lot and Space Manifest
* Perform duplication processing
* Perform bit integrity checks
* Capture and maintain storage statistics

These activities can be seen directly in the Mill database, which maintains a table for each of the items listed above (with distinct tables for audit and manifest entries.)

#### Duplication

When a file is added or updated in DuraCloud, a part of the initial processing determines if that file is stored in a space which should be duplicated to a secondary (or tertiary) storage provider. Each DuraCloud account can be connected to multiple storage providers. There is always a primary provider, which is defined in the Management Console configuration. The primary provider is where content is stored initially by default. 

Each account with multiple providers also has a **Duplication Policy** which defines the spaces which should be duplicated across providers. Most often all spaces in these accounts are duplicated, but not in all cases. The Duplication Policy defines, for an account, which spaces to duplicate, and to which providers. When a file is updated, this policy is checked.

For each change made to a file in a duplicated space, a message is placed on a **Duplication Queue**. The Mill worker processes this message by duplicating these changes to the secondary provider.

As with any distributed system, this process will occasionally fail for one or more files. Retry logic often catches and allows the work to complete successfully. However, as a means to check for and handle any missed duplication events, a **Duplication Producer** process can be executed at any time. This process adds all items in all spaces with a duplication policy to a "low priority" duplication queue (the primary duplication queue is considered "high priority"). The actions performed for messages on the low priority queue are the same as for the high priority queue, though the vast most messages result in a verification, but no further action.

#### Bit Integrity

A major component of why DuraCloud is considered valuable by customers is because of its ability to perform a third-party audit on content in cloud storage. This work is performed by the **Bit Integrity Producer** and the Bit Integrity workers. The Bit Integrity Producer, in similar fashion to the Duplication Producer, adds a **Bit Integrity Task** to the **Bit Integrity Queue** for every file in the DuraCloud system. 

*The exception to this rule is that content in Glacier storage providers are not included in bit integrity checks due to additional time and cost required to request and retrieve each file.*

For each file added to the Bit Integrity Queue, a worker process retrieves the file from storage, computes the MD5 checksum of the file, then compares that checksum to the Space Manifest and the checksum provided by the storage provider. If any of these checks fail, the file is retried. If it fails again, it is added to a failure list.

After all of the content in a space has been checked, a message is added to the **Bit Integrity Report Queue**. This message is processed by a **Bit Integrity Report Worker** to determine if there were any failures (for which a notification is sent to DuraCloud system administrators) and to create a final report for the space.

#### Storage Statistics

It is important for users and administrators alike to know how much content is contained in each DuraCloud space and account. Capturing this information is the purpose of the **Storage Stats Producer**. This process is similar to other producers; it adds a message to the **Storage Stats Queue** for each space in the DuraCloud system.

The **Storage Stats Worker** then queries for  total files and total bytes stored in each space. How these queries are performed depend on the storage provider type. For Amazon S3, the data is retrieved as AWS CloudWatch metrics. These metrics are made available on a daily basis. For Amazon Glacier no such metrics are available, so the data is queried from the DuraCloud Mill database. For OpenStack-based storage providers the data is available through the OpenStack API (as it should be for S3 and Glacier, but alas, we work with what we are given.)