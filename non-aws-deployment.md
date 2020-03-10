# Deploying DuraCloud without AWS

While DuraCloud was initially developed to be run within an Amazon Web Services (AWS) environment, it is also possible to run DuraCloud outside of the AWS context. The sections below describe how to swap out some or all of the DuraCloud deployment environment to non-AWS alternatives.

Thanks to the team at the University of Toronto for their development contributions and to [CANARIE](https://www.canarie.ca/) for funding the work required to make these features possible. :thumbsup:

## Versions
Ensure that you are running at least the following versions of DuraCloud software:
* DuraCloud version 7.0+
* DuraCloud Management Console version 6.0+
* DuraCloud Mill 4.0+

If you are upgrading from a previous version of DuraCloud, ensure that you have run the schema update script `resources/sql/update-non-aws.sql` in the Management Console Baseline. This will add required columns to the `duracloud_mill` and `global_properties` tables in the accounts database. The exact command will vary depending on your MySQL installation.
```
mysql <accounts_db_name> < resources/sql/update-non-aws.sql
```
## Email
SMTP can be used in place of AWS SES for sending emails. Setting up an SMTP server is beyond the scope of this document, but any standard SMTP setup should work fine, with or without authentication.

To start using SMTP to send emails, add the following to the `.properties` files used to configure each DuraCloud application:
```
notification.type=smtp
notification.host=your.smtp.server.com
notification.port=25 # Defaults to 25 if not specified
```
If your SMTP server requires authentication, also add
```
notification.user=smtp_username
notification.pass=smtp_password
```

## Queues
RabbitMQ can be used in place of SQS to provide system queues. This was developed and tested using RabbitMQ version 3.7.17, Erlang 22.0.7. The `rabbitmq_message_timestamp` plugin must be installed. SQS functions are provided by a RabbitMQ `direct` exchange

### Replacing SQS
1. In RabbitMQ, create an Exchange of type `direct`
1. In RabbitMQ server, create the following queues (the actual names are arbitrary, but make sure they are unique):
  * audit
  * bit-integrity
  * dup-high-priority
  * dup-low-priority
  * bit-report
  * dead-letter
  * storagestats
1. Bind all queues created in step 2 to the exchange created in step 1, use the queue names as the routing key
1. In Management Console > root console > Duracloud MiIl:
  * Change Queue Type to RabbitMQ
  * Input all the config, including the direct exchange name created in step 1 and audit queue name created in step 2.
  * Save settings and restart Tomcat

## Notifications
RabbitMQ can be used in place of SNS to provide system notifications. SNS functions are provided by a RabbitMQ `fanout` exchange.

### Replacing SNS
1. In RabbitMQ server, create an Exchange (type `fanout`)
1. In Management Console > root console > Global Properties:
  * Change Queue Type to “RabbitMQ”
  * Input all the config, including the fanout exchange name created in step 1
  * Save settings and restart Tomcat

## Updating the Mill
To allow the Mill to connect to RabbitMQ, either for Queues, Notifications, or both, add these extra configuration options to your Mill `.properties` file.
```
queue.type=rabbitmq
rabbitmq.host=rmq_host
rabbitmq.port=5672
rabbitmq.vhost=your_vhost
rabbitmq.exchange=<same as the "direct" exchange>
rabbitmq.username=rmq_username
rabbitmq.password=rmq_password
```
Ensure that all of the `queue.name.*` parameters match the names of the queues you created in RabbitMQ.

## OpenStack Swift Storage
OpenStack Swift can be used as a DuraCloud Storage Provider, regardless of whether other components in the system use non-AWS options. These configurations are managed in the DuraCloud Management Console along with all other Storage Provider types.

To use Swift as a DuraCloud Storage provider, you must:
* be running OpenStack Swift Train or newer
* install the S3 middleware API
* create EC2 credentials for the domain/project where you want to store audit logs/duplication policies

### Swift for System Storage
OpenStack Swift can also be used in place of Amazon S3 for background DuraCloud system storage.

To use Swift as a storage backend for duplication policies and audit logs, you must have OpenStack configured such that it can be used as a DuraCloud Storage Provider (see list above). You must also include the following fields in your `.properties` files for both DuraStore/DurAdmin and the Mill.
```
swift.accessKey=swiftaccesskey
swift.secretKey=swiftsecretkey
swift.endpoint=https://s3.example.com
swift.signerType=signer_type_if_required
```

With these settings in place:
* Durastore will read audit logs from the audit log bucket (in Swift) that you have configured in the Management Console.
* Workman and the Loopingduptaskproducer will read duplication policies from the container configured in Mill `.properties`.
* The Audit Log Generator will write audit logs to the audit log bucket configured in Mill `.properties`.

It is possible to use separate credentials (and therefore, separate domains/projects) for audit logs and duplication policies, by using a different `.properties` file for each.