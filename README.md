# Serverless framework

## serverless.yml

From our previous classes, we learned that serverless.yml is the file where we define our **AWS Lambda Functions**, the events that trigger them and any AWS infracstructure resources they require.

We created `serverless.yml` to satisfy the requirements of our task and im going to walk you through our code. 


### Root properties:

```yml
service: schedule-function-2

frameworkVersion: '3'
```

### Parameters
We used parametars to adapt the configuration based on the stage,

```yml

# Parameters
params:
  dev:
    rate: rate(10 minutes) # trigger event every 2 minutes
#    cron: cron(0/10 * * * ? *)
  research:
    rate: rate(15 minutes) # trigger event every 5 minutes
#    cron: cron(0/15 * * * ? *)
```

We can create rules that self-trigger on an automated schedule in CloudWatch Events using cron or rate expressions. Meaning, if the function is deployed in `--stage dev` and has a variable that refers to our rate parameter, as we have in our `hello` function, it will be self-triggered for the exact amount of time we have defined in our rate expression.

### Provider

Here we will define our general settings. Such as the name of the platform we are working on, our region etc.

```yml
provider:
  name: aws
  region: eu-west-1 
  profile: mentorship
  
```

*Defining `profile` allowed me for further execution of the serverless command, in my terminal, without specifing `--aws-profile` in the syntax.*

When we invoke a function and we dont have specified resources for it, that function will get our default resources from the general function settings.

```yml
  runtime: nodejs14.x
  memorySize: 256
  timeout: 3
  stage: dev
```



### IAM Role
Creating IAM role in `provider` will automaticlly attach to every function in this template.

```yml
  iam:
    role:
      statements:
# log group permissions	  
        - Effect: Allow
          Action:
            - logs:CreateLogGroup
            - logs:CreateLogStream
            - logs:PutLogEvents
          Resource:
            - 'Fn::Join':
              - ':'
              -
                - 'arn:aws:logs'
                - Ref: 'AWS::Region'
                - Ref: 'AWS::AccountId'
                - 'log-group:/aws/lambda/*:*:*'
# s3 permissions
        - Effect: 'Allow'
          Action:
            - 's3:ListBucket'
          Resource:
            Fn::Join:
              - ''
              - - 'arn:aws:s3:::'
                - Ref: ServerlessDeploymentBucket
        - Effect: 'Allow'
          Action:
            - 's3:PutObject'
          Resource:
            Fn::Join:
              - ''
              - - 'arn:aws:s3:::'
                - Ref: ServerlessDeploymentBucket
                - '/*'
```

```yml
# functions
functions:
  hello:
    handler: func0.hello
    runtime: nodejs14.x
    memorySize: 128
    timeout: 2
    name: ${opt:stage}-hello
    description: Hello from Lambda!
    architecture: x86_64
    events:
      - schedule: ${param:cron} 

# Trigger: API Gateway
  currentTime:
    handler: func1.endpoint
    name: ${opt:stage}-CurrentTime
    description: What's the time?
    events:
      - httpApi:
          path: /time
          method: get

# # S3 Event
  s3event:
    handler: func2.upload
    name: ${opt:stage}-s3event
    description: Upload file to trigger the function
    events:	
      - s3:
          bucket: ${opt:stage}-s3bucket1 
          event: s3:ObjectCreated:*
          rules:
            - suffix: .jpg
 ```

