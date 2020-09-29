# Service

## Security

- Security is job 0 for everyone. What do I need to do to make sure my service is configured securely
- Encryption (options)
  - At rest
    - Encrypt Kinesis Data Streams (KDS) using AWS Key Management Service Customer Master Key (AWS KMS CMK) to simplify the data encryption process and meet stringent data management requirements.
      - AWS KMS CMK is used to encrypt the data for a stream as it is written/read from the storage layer. Thus, the data in the KDS remains encrypted at rest.
      - Recommend using AWS KMS CMK from the beginning of a Kinesis Data Stream creation to assure all data is encrypted at rest from the start. Enabling AWS KMS CMK after a Kinesis Data Stream is created will encrypt new data but it will not encrypt existing data.
      - The default CMK used by KMS is free, although any API calls to the AWS KMS service will incur costs.
      - There is also an option to use your own master keys using KMS, this is the recommended option if more flexibility is required such as using a KSM master key that is located on a different account than where the Kinesis Data Stream resides.
        - When using a user generated KMS Master Keys, note that the IAM principals for the producers and consumers must be included in the KMS master key policy with Encrypt/Decrypt permissions, to allow read/write from and to an encrypted stream. Failure to do so could result in data loss or application malfunction.
        - Use CloudTrail to monitor API calls to your KDS, to determine who made requests, from where (IP address) and for which stream.
  - In transit
    - Utilize Kinesis VPC endpoints to prevent traffic from leaving the Amazon network between your VPC and your KDS implementation.
      - Determine the granularity of your VPC policy for your KDS which can be, for example:
        - Read-only access.
        - Access restricted to a specific Kinesis Data Stream.
        - Access restricted to a specific Kinesis Data Stream from only a specific VPC endpoint.
      - Create IAM policies to determine what groups/users are allowed to do or not on a specific Kinesis Data Stream. The policies should follow the least privilege principle, allowing only read, write or both for a specific stream, a set of streams or all streams in an account (e.g. Administrators may have access to all streams in an account, depending on your policy).
      - Use IAM roles with these policies to manage temporary credentials for producer and client applications in order to access KDS.
    - It is recommended that clients consuming KDS support TLS 1.2 or later.
      - Clients must also support cipher suites with Perfect Forward Secrecy (PFS) such as:
        - Ephemeral Diffie-Helman (DHE).
        - Elliptic Curve Ephemeral Diffie-Hellman (ECDHE).
    - Requests must also be signed with the access key id and secret access key for the IAM principal making the call
    - AWS Security Token Service is also a supported option that can be used to generate temporary user security credentials that in turn can be used to sign requests.
- FIPS endpoints for Kinesis Data Streams:
  - kinesis-fips.us-east-1.amazonaws.com
  - kinesis-fips.us-east-2.amazonaws.com
  - kinesis-fips.us-west-1.amazonaws.com
  - kinesis-fips.us-west-2.amazonaws.com
  - *For the most up to date FIPS endpoints information, see the official docs: <https://aws.amazon.com/compliance/fips/>
- Prisma rules affecting this service
- Default least privilege IAM permissions
- For examples of policies that can be applied to Kinesis Data Streams see the following documentation:
  - <https://docs.aws.amazon.com/streams/latest/dev/controlling-access.html#kinesis-using-iam-examples>

## Reliability

- Limits (will link to documentation)
  - There are two main limits to consider:  
    - KDS Control Plane API limits for creation and management of your data streams.
    - KDS Data Plane API limits used for collecting and processing your data records in real time.
    - KDS limits are also supported by the Quotas service, for more information see the AWS docs:
      - <https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html>

- How do you use this service in a resilient fashion
  
  - Handling failures
    - For task failures, that are managed by a worker, let the worker start a new processor to handle the shard.
    - For application failures, detect the failure and restart the application. Once the application starts up again, a new worker will be instantiated which will start a new record processor with new shards automatically assigned to it.
    - For EC2 instances hosting an application that fails, ensure that there are multiple EC2 instances being managed as part of an Auto Scaling Group, so new EC2 instances can be automatically launched to create new record processors to handle the shards that are no longer being processed by the failed EC2 instances.
  
  - Handling duplicate records
    - For Producer retries, a primary key should be embedded with the record to allow for deduplication during processing of data records.
    - For Consumer retries, strive to make the writes of duplicate data idempotent by using sequence checkpoints and primary keys.
  
  - Handling startups
    - Change the default behavior of the record processors by setting the initialPositionInStream to "TRIM_HORIZON" as this will allow the application to always read from the beginning of the stream, thus allowing your application to process the older data in the stream first.
    - Use the Kinesis Client Library (KCL), to handle scenarios where data may be processing at a faster rate than the allowed limit, since the KCL will handle for you any throttling or exceptions that may occur.

- High Availability considerations (application and infrastructure)
  - Utilize Availability Zones to distribute your instances across physically redundant data centers.
  - Create an Auto Scaling Group to run your EC2 instances that will in turn host the Kinesis application instances. In this manner, if an EC2 instance fails, the Auto Scaling group will launch a new EC2 instance to replace the one that failed.
    - Ensure that you configure your EC2 instances to launch your KDS application when they start so that your application instance be replaced after an EC2 failure.
  
- How to test
  - Generate test data by using the Amazon Kinesis Data Generator (KDG)
    - <https://awslabs.github.io/amazon-kinesis-data-generator/web/help.html>
- Deployment considerations (canary, blue-green, what is approved)

## Performance

- How can you operate with this service at scale?
  - In order to avoid hitting the limit of 5 GetRecords calls per second, you should poll each shard at an interval of one call per second per application. The KCL polling setting is set to 1 second by default, as this is considered a best practice.
  - Utilize enhanced fan-out to support multiple consumers of a stream with an increased dedicated throughput of 2 MB of data per second per shard. Consumers that use the enhanced fan-out feature do not have to contend with other consumers currently receiving data from a stream. Some additional configuration is required to enable and manage this feature.

## Cost Optimization

- What are some of the tradeoffs for performance vs cost
  - Be aware that splitting a shard will increase the number of shards in your stream and therefore increase the data capacity of the stream. However, the trade off is that since you are charged on a per-shard basis, conducting a split of a shard will also increase the cost of your stream. On the other hand, when you merge shards together, this action reduces the number of shards in your stream which also decreases the data capacity and cost of the stream. See metrics/monitoring for information on metrics that can signal when a shard split or a merge might be needed.
  - Once a KDS application has finished processing data, you should terminate the EC2 instance to avoid additional costs.

## Operations

- Minimum monitoring and specific metrics to have alarms on
  - Enable Basic monitoring to analyze stream-level metrics which are sent every minute automatically at no charge.
  - Enable CloudTrail to monitor on-host CPU metrics and the amount of memory consumed. Additionally, capturing agent error counters will serve to identify resource usage for your data producers, which can provide insights into possible configuration or host errors.
  
- Scaling metrics/monitoring
  - Enable Enhanced monitoring to analyze shard-level metrics. The enhanced metrics can be singularly selected and include: incoming and outgoing bytes and records, write and read throughput exceptions, the age of the iterator in milliseconds. Or you can also enable 'ALL' of these metrics.
  - Monitor the IncomingBytes and OutgoingBytes CloudWatch metrics to track the shard usage and utilize the results to compare to the number of shards in the stream and setup an alarm when reaching a threshold.
  - Monitor the GetRecords.IteratorAgeMilliseconds CloudWatch metric to alert you if applications are keeping up with the data coming in and setup an alarm.

## Teams already using this service
