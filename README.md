# Serverless framework
![Visitors](https://api.visitorbadge.io/api/visitors?path=https%3A%2F%2Fgithub.com%2Fx0rCTF%2Fserverless.yml&countColor=%23263759&style=flat-square)


## serverless.yml

In our previous classes, we learned that serverless.yml is the file where we define our **AWS Lambda Functions**, the events that trigger them and any AWS infracstructure resources they require.

We created `serverless.yml` to meet the requirements of our task and I am going to walk you through our code. 


### Root properties:
These are the default root properties and we did not change anything here.
```yml
service: schedule-function-2

frameworkVersion: '3'
```

### Parameters
We used parameters to adapt the configuration based on the stage.

```yml

# Parameters
params:
  dev:
    rate: rate(10 minutes) # trigger event every 10 minutes
#    cron: cron(0/10 * * * ? *)
  research:
    rate: rate(15 minutes) # trigger event every 15 minutes
#    cron: cron(0/15 * * * ? *)
```

We can create rules that self-trigger on an automated schedule in CloudWatch Events using cron or rate expressions. Meaning, if the function is deployed with `--stage dev` and has a variable that refers to our rate parameter, as we have in our `hello` function, it will be self-triggered for the exact amount of time we have defined in our rate expression.

### Provider

This is where our general settings are defined.

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
Creating an IAM role within `provider` will automaticlly attach itself to every function in this template.

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
      - schedule: ${param:rate} 

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

