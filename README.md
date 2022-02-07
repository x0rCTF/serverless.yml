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

We can create rules that self-trigger on an automated schedule in CloudWatch Events using cron or rate expressions. Meaning, if the function is deployed in `--stage dev` and has a variable that refers to our rate parameter, it will be self-triggered for the exact amount of time we have defined in our rate expression.

### Provider

Here we'll define our general settings. Such as the name of the platform we are working on, our region and profile etc.

```yml
provider:
  name: aws
  region: eu-west-1 
  profile: mentorship
  
```

*Defining `profile` allowed me for further execution of the serverless command, in my terminal, without specifing `--aws-profile` in my syntax.*

```yml
  runtime: nodejs14.x
  memorySize: 256
  timeout: 3
  stage: dev
```
