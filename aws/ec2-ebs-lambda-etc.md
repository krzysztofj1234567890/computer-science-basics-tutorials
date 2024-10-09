# AWS Lambda

Lambda runs your code on a high-availability compute infrastructure and performs all of the administration of the compute resources, including server and operating system maintenance, capacity provisioning and automatic scaling, and logging.

The Lambda service runs your function only when needed and scales automatically. You only pay for the compute time.

Lambda is an ideal compute service for application scenarios that need to scale up rapidly, and scale down to zero when not in demand. 

Lambda functions run in a Lambda-managed VPC that has internet access

For example:
- __File processing__: Use Amazon Simple Storage Service (Amazon S3) to trigger Lambda data processing in real time after an upload.
- __Stream processing__: Use Lambda and Amazon Kinesis to process real-time streaming data for application activity tracking
- __Web applications__: Combine Lambda with other AWS services to build powerful web applications that automatically scale up and down
- __IoT backends__: Build serverless backends using Lambda to handle web, mobile, IoT

Features:
- Use __environment variables__ to adjust your function's behavior without updating code.
- Manage the deployment of your functions with __versions__, so that, for example, a new function can be used for beta testing
- Create a __container image__ for a Lambda function by using an AWS provided base image
- Package __libraries__ and other dependencies to reduce the size of deployment archives
- Add a dedicated __HTTP(S) endpoint__ to your Lambda function.
- Configure your Lambda function URLs to __stream response payloads__ back to clients from Node.js functions

## Example of lambda function

```
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    
    # Get the length and width parameters from the event object. The 
    # runtime converts the event object to a Python dictionary
    length = event['length']
    width = event['width']
    
    area = calculate_area(length, width)
    print(f"The area is {area}")
        
    logger.info(f"CloudWatch logs group: {context.log_group_name}")
    
    # return the calculated area as a JSON string
    data = {"area": area}
    return json.dumps(data)
    
def calculate_area(length, width):
    return length*width
```

handler function is always the entry point to your code

The function __lambda_handler__ takes two arguments, event and context. An __event__ in Lambda is a JSON formatted document that contains data for your function to process.
If your function is invoked by another AWS service, the event object contains information about the event that caused the invocation. For example, if an Amazon Simple Storage Service (Amazon S3) bucket invokes your function when an object is uploaded, the event will contain the name of the Amazon S3 bucket and the object key.

The second argument your function takes is __context__. Lambda passes the context object to your function automatically. The context object contains information about the function invocation and execution environment.

With Python, you can use either a print statement or a Python logging library to send information to your function's log. 

You can invoke the Lambda function using console. Just create an event and run a test.

## lambda application

Includes lambda and all other components you need to run it. For example:
- An S3 bucket for users to upload PDF files to
- A Lambda function in Python which reads the uploaded file and creates an encrypted, password-protected version of it
- A second S3 bucket for Lambda to save the encrypted file in
- AWS Identity and Access Management (IAM) policy to give your Lambda function permission to perform read and write operations on your S3 buckets.

You can deploy your app manually by creating and configuring resources with the AWS Management Console or the AWS Command Line Interface (AWS CLI). You can also deploy the app by using the AWS Serverless Application Model (AWS SAM). AWS SAM is an infrastructure as code (IaC) tool. With IaC, you don’t create resources manually, but define them in code and then deploy them automatically.

## Concepts

- __Function__: A function is a resource that you can invoke to run your code in Lambda. A function has code to process the events that you pass into the function or that other AWS services send to the function.
- __Trigger__: A trigger is a resource or configuration that invokes a Lambda function. Triggers include AWS services that you can configure to invoke a function and event source mappings. An event source mapping is a resource in Lambda that reads items from a stream or queue and invokes a function.
- __Event__: An event is a JSON-formatted document that contains data for a Lambda function to process.
```
{
  "TemperatureK": 281,
  "WindKmh": -3,
  "HumidityPct": 0.55,
  "PressureHPa": 1020
}
```
- An __execution environment__ provides a secure and isolated runtime environment for your Lambda function. An execution environment manages the processes and resources that are required to run the function.
- You deploy your Lambda function code using a __deployment package__. Lambda supports two types of deployment packages: zip and container.
- The __runtime__ provides a language-specific environment that runs in an execution environment. Example: python3.12, python3.11, java21
- A Lambda __layer__ is a .zip file archive that can contain additional code or other content. A layer can contain libraries, a custom runtime, data, or configuration files.
- When you invoke or view a function, you can include a __qualifier__ to specify a version or alias. A version is an immutable snapshot of a function's code and configuration that has a numerical qualifier. For example, my-function:1
- A __destination__ is an AWS resource where Lambda can send events from an asynchronous invocation. 

## Programing environment

When the handler finishes processing the first event, the runtime sends it another. The __function's class stays in memory__, so clients and variables that are declared outside of the handler method in initialization code can be reused.

Your function also has access to local storage in the __/tmp__ directory. The directory content remains when the execution environment is frozen, providing a transient cache that can be used for multiple invocations. 

When AWS __X-Ray__ tracing is enabled, the runtime records separate subsegments for initialization and execution.

The runtime captures __logging__ output from your function and sends it to __Amazon CloudWatch Logs__

Lambda __scales__ your function by running additional instances of it as demand increases, and by stopping instances as demand decreases. Incoming requests might be processed __out of order__ or __concurrently__.

## Configuring AWS Lambda functions

- __memory__: Memory is the amount of memory available to your Lambda function at runtime. You can increase or decrease the memory and CPU power allocated to your function using the Memory setting. 
```
# Configuring function memory in AWS SAM
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: An AWS Serverless Application Model template describing your function.
Resources:
  my-function:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .
      Description: ''
      MemorySize: 1024
      # Other function properties...
```
- __cpu__: Lambda allocates CPU power in proportion to the amount of memory configured. At 1,769 MB, a function has the equivalent of one vCPU.
- __ephemeral storage__. Lambda provides ephemeral storage for functions in the /tmp directory. You can control the amount of ephemeral storage allocated to your function using the Ephemeral storage setting. You can configure ephemeral storage between 512 MB and 10,240 MB. All data stored in /tmp is encrypted at rest with a key managed by AWS
  - Extract-transform-load (ETL) jobs
  - Machine learning (ML) inference: Many inference tasks rely on large reference data files, including libraries and models. With more ephemeral storage, you can download larger model
```
# Configuring ephemeral storage (AWS SAM)
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: An AWS Serverless Application Model template describing your function.
Resources:
  my-function:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: .
      Description: ''
      MemorySize: 128
      Timeout: 120
      Handler: index.handler
      Runtime: nodejs20.x
      Architectures:
        - x86_64
      EphemeralStorage:
        Size: 10240
      # Other function properties...
```
- Giving Lambda functions access to resources in an __Amazon VPC__
- You can configure your Lambda function URLs to __stream response__ payloads back to clients. Response streaming can benefit latency sensitive applications by improving time to first byte (TTFB) performance.

## Invocation

When you invoke a function, you can choose to invoke it __synchronously__ or __asynchronously__. With synchronous invocation, you wait for the function to process the event and return a response. With asynchronous invocation, Lambda queues the event for processing and returns a response immediately. The InvocationType request parameter in the Invoke API determines how Lambda invokes your function. A value of __RequestResponse indicates synchronous invocation__, and a value of __Event indicates asynchronous invocation__.

If the function invocation results in an __error__, for synchronous invocations, view the error message in the response and retry the invocation manually. For asynchronous invocations, Lambda handles retries automatically and can send invocation records to a destination.

Asynchronous is triggered by: S3, SNS. Errors can be sent to SQS or EventBridge

__Event filtering__: control which records from a stream or queue Lambda sends to your function. For example, you can add a filter so that your function only processes Amazon SQS messages containing certain data parameters. Event filtering works only with certain event source mappings (SQS, MSK,Kinesis, DynamoDB)

__Retries__: determine the strategy for handling errors related to your function code. Lambda does not automatically retry these types of errors on your behalf. To retry, you can manually re-invoke your function, send the failed event to a queue for debugging, or ignore the error. 

__Invocation methods__:
- AWS Console
- The Invoke API
- CLI
- function URL HTTP(S) endpoint
```
https://url-id.lambda-url.us-east-1.on.aws
```
- event that occurs elsewhere in your application

## Scaling

Concurrency is the number of in-flight requests that your AWS Lambda function is handling at the same time. For each concurrent request, Lambda provisions a separate instance of your execution environment. As your functions receive more requests, Lambda automatically handles scaling the number of execution environments until you reach your account's concurrency limit. By default, Lambda provides your account with a total concurrency limit of 1,000 concurrent executions across all functions in an AWS Region.


