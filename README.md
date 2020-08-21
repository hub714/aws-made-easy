# Well-Architected Guidance for Services

The purpose of this repo is to provide specific guidance on commonly used services at [customer]. The recommendations serve as a baseline and should be tailored by the individual customer teams.

The general structure is as follows:

## Service
### Security
- Security is job 0 for everyone. What do I need to do to make sure my service is configured securely
- Encryption (options)
  - At rest
  - In transit
- FIPS endpoints?
- Corporate rules affecting this service
- Default least privilege IAM permissions
### Reliability
- Limits (will link to documentation)
- How do you use this service in a resilient fashion
- High Availability considerations (application and infrastructure)
- How to test
- Deployment considerations (canary, blue-green, what is approved)
### Performance
- How can you operate with this service at scale?
### Cost Optimization
- What are some of the tradeoffs for performance vs cost
### Operations
- Minimum monitoring and specific metrics to have alarms on
- Scaling metrics/monitoring

## Teams already using this service

***
## Adding additional services
To add a new service, copy the template folder and create a new service:
> cp -R template service

## Initial target services
- EC2
- S3
- Kinesis
- SQS
- SNS
