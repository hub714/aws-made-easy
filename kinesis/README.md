# Service
## Security
- Security is job 0 for everyone. What do I need to do to make sure my service is configured securely
- Encryption (options)
  - At rest
    - Encrypt Kinesis Data Streams using AWS Key Management Service Customer Master Key (AWS KMS CMK) to simplify the data encryption process and meet stringent data management requirements.
      - AWS KMS CMK is used to encrypt the data for a stream as it is written/read from the storage layer. Thus, the data in the Kinesis Data Streams remains encrypted at rest.
      - Recommend using AWS KMS CMK from the beginning of a Kinesis Data Stream creation to assure all data is encrypted at rest from the start. Enabling AWS KMS CMK after a Kinesis Data Stream is created will encrypt new data but it will not encrypt existing data.
      - The default CMK used by KMS is free, although any API calls to the AWS KMS service will incur costs.
      - There is also an option to use your own master keys using KMS, this is the recommended option if more flexibility is required such as using a KSM master key that is located on a different account than where the Kinesis Data Stream resides.
        - When using a user generated KMS Master Keys, note that the IAM principcals for the producers and consumers must be included in the KMS master key policy with Encrypt/Decrypt permissions, to allow read/write from and to an encrypted stream. Failure to do so could result in data loss or application malfunction.
        - Use CloudTrail to monitor API calls to your Kinesis Data Streams, to determine who made requests, from where (IP address) and for which stream.
  - In transit
    - Utilize Kinesis VPC endpoints to prevent traffic from leaving the Amazon network between your VPC and your Kinesis Data Streams implementation.
      - Determine the granularity of your VPC policy for your Kinesis Data Streams which can be, for example:
        - Read-only access
        - Access restricted to a specific Kinesis Data Stream
        - Access restricted to a speicici Kinesis Data Stream from only a specific VPC endpoint
      - Create IAM policies to determine what groups/users are allowed to do or not on a specific Kinesis Data Stream. The policies should follow the least privilege principle, allowing only read, write or both for a specifc stream, a set of streams or all streams in an account (e.g. Administrators may have access to all streams in an account, depending on your policy).
      - Use IAM roles with these policies to manage temporary credentials for producer and client applications in order to access Kinesis Data Streams.
    - It is recommended that clients consuming Kinesis Data Streams support TLS 1.2 or later.
      - Clients must also support cipher suites with Perfect Forward Secrecy (PFS) such as:
        - Ephemeral Diffie-Helman (DHE)
        - Elliptic Curve Ephemeral Diffie-Hellman (ECDHE)
    - Requests must also be signed with the access key id and secret access key for the IAM principal making the call
    - AWS Security Token Service is also a supported option that can be used to generate temporary user security credentials that in turn can be used to sign requests.
- FIPS endpoints?
- Prisma rules affecting this service
- Default least privilege IAM permissions
## Reliability
- Limits (will link to documentation)
  - https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html
- How do you use this service in a resilient fashion
- High Availability considerations (application and infrastructure)
- How to test
- Deployment considerations (canary, blue-green, what is approved)
## Performance
- How can you operate with this service at scale?
## Cost Optimization
- What are some of the tradeoffs for performance vs cost
## Operations
- Minimum monitoring and specific metrics to have alarms on
- Scaling metrics/monitoring

## Teams already using this service