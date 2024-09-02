---
title: "Localstack: Running AWS Locally"
seoTitle: "Localstack: Running AWS Locally"
seoDescription: "Run AWS without a surprise AWS Cloud Bill!"
datePublished: Sat Aug 31 2024 06:48:29 GMT+0000 (Coordinated Universal Time)
cuid: cm0hs7eoi001708le7oisd4u7
slug: localstack-running-aws-locally
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1725086879729/9d263540-1542-466a-82ba-b4cee535508e.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1725086899713/2d5fcfce-6d0f-4bdf-87ff-fd4d53f957c5.jpeg
tags: aws, testing, localstack

---

Most of us learn AWS products such as S3, DynamoDB etc. We open an AWS account. Then we give our credit card. Then we spin up resources. Finally we are done with work and **forget** to switch off the resources that we have set up. Alas, the bill arrives and there is regret.

## Introducing Localstack

Localstack is a simulation of AWS environment that can be done locally using Docker. You pull Docker images and run them and get AWS in your local machine.

You can then use it to understand the AWS APIs like S3's `GET`, `PUT`, `DELETE` Bucket; `Create`, `Update`, `Delete` on DynamoDB; `Create`, `Delete`, `Update`, `DLQ` for SQS etc.

This allows to get the hang of AWS Services without utilising AWS directly. This is especially helpful in organisations with 1000s of developers where cost of learning AWS services can shoot up dramatically.

## Code Example

You can follow/see code on setting up Localstack in my Github repo [here.](https://gitpod.io/#https://github.com/aymanapatel/localstack-aws)

A simple `init.sh` and `start.sh` is all it takes to get Localstack running in Docker.

```bash
echo "Pulling Docker images ..."
docker pull localstack/localstack
pip install localstack awscli awscli-local
docker pull mlupin/docker-lambda:nodejs14.x &
docker pull lambci/lambda:ruby2.7 &
docker pull lambci/lambda:python3.7 &

# start LocalStack container in the background
echo "Starting LocalStack instance ..."
DEBUG=1 localstack start -d
```

After Localstack is running you can call the \`aws\` CLI (Local):

```bash
/workspace/localstack-aws (main) $ awslocal s3 mb s3://ayman-bucket
make_bucket: ayman-bucket

/workspace/localstack-aws (main) $ awslocal s3 ls
2022-06-31 06:40:21 ayman-bucket
```

Running via a AWS Native SDK is also possible:

```javascript
const { S3Client, ListBucketsCommand } = require('@aws-sdk/client-s3');

const s3 = new S3Client({
  region: 'us-east-1',
  endpoint: 'http://<LOCAL_ENDPOINT>:4566',
  credentials: {
    accessKeyId: 'test',
    secretAccessKey: 'test',
  },
});

// Call an S3 API using the LocalStack endpoint
s3.send(new ListBucketsCommand({}))
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

Shell Response:

```bash
/workspace/localstack-aws/languages/nodejs (main) $ node aws-s3.js 
{
  '$metadata': {
    httpStatusCode: 200,
    requestId: 'd415023f-68a4-4af2-8e88-f6375f76b343',
    extendedRequestId: 's9lzHYrFp76ZVxRcpX9+5cjAnEH2ROuNkd2BHfIa6UkFVdtjf5mKR3/eTPFvsiP/XV/VLi31234=',
    cfId: undefined,
    attempts: 1,
    totalRetryDelay: 0
  },
  Buckets: [ { Name: 'ayman-bucket', CreationDate: 2024-08-31T06:40:21.000Z } ],
  Owner: {
    DisplayName: 'webfile',
    ID: '75aa57f09aa0c8caeab4f8c24e99d10f8e7faeebf76c078efc7c6caea54ba06a'
  }
}
```

## Features

1. Simulate AWS features such as S3, DynamoDB, SQS, SNS, RDS. Feature coverage for Free and Paid can be found [here](https://docs.localstack.cloud/user-guide/aws/feature-coverage/)
    
2. Security testing using Role policies of AWS IAM(Identity and Access Management).
    
3. Chaos Engineering: Run Chaos Engineering tests to simulate outages by injection faults, network degredation using the [Chaos API](https://docs.localstack.cloud/user-guide/aws/feature-coverage/)
    
4. You can also simulate [Snowflake in Localstack](https://www.localstack.cloud/localstack-for-snowflake). This increases the scope of Localstack to be beyond just AWS simulation.
    

### Localstack GUI

Localstack Docker version is all CLI based which makes it harder to navigate if you are a large number S3, Lambda etc resources. [Localstack Cloud](https://app.localstack.cloud/) has a GUI interface to manage this.

## Test Containers + Localstack = Match made in heaven

Test Containers is one of those technologies that is gaining steam as it emphasis on integration environment. Localstack and Testcontainers can be leveraged to run AWS test cases locally which can aid in following the Test pyramid and help catch bugs faster and earlier.

* [Check out Testcontainer + Localstack Documentation](https://java.testcontainers.org/modules/localstack/)
    

## Caveats

1. **Maintaining Dev/Prod Parity:** Since Localstack is not an AWS organization, their will be some implementation level details. Localstack will try to match the behaviour, but their will still be some nuanced difference as it is a seperate environment when compared to AWS.
    
2. **New AWS Features will not be their on day 1:** New AWS feature like the new [S3 conditional write](https://aws.amazon.com/about-aws/whats-new/2024/08/amazon-s3-conditional-writes/) will take time to be available on Localstack.
    
3. **Free version does not have all AWS Features:** The most common features such as S3, DynamoDB, SQS and SNS are present in free version; but some items such as Database (RDS/Aurora) are in Pro image.
    

It is always good to know what you are leveraging Localstack for. If you want to learn the services API such as S3, Dynamo, SQS; then Localstack is a great platform. But if you want to see actual usage or want to see how AWS manages things at scale, it is better to go for AWS Account only.

Hence, use Localstack when you are in learning phase and checkout my [Github repo](https://github.com/aymanapatel/localstack-aws) with some simple scripts, PoCs on Localstack.