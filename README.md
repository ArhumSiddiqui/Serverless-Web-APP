# Serverless-Web-APP

This project is a serverless reminder application built using AWS services, including Lambda, Step Functions, API Gateway, Simple Email Service (SES), and a static frontend. The application is designed to automate email reminders and provide users with real-time interactions via an API and frontend interface. This is based off of Adrian’s serverless web app tutorial.

## Features

- Serverless architecture for efficient scalability and cost-effectiveness.
- Automatic email notifications using AWS Simple Email Service (SES).
- State machine workflow to manage and process reminders.
- Fully managed API integration using AWS API Gateway and Lambda.
- Static frontend for easy user interaction with the reminder application.

## Architecture

The application architecture consists of the following components:

- **API Gateway**: Fronts the serverless API for user interactions.
- **Lambda Functions**: Handles email sending, API logic, and triggers for state machine actions.
- **Step Functions**: Core state machine logic for managing reminder workflows.
- **Simple Email Service (SES)**: Sends email notifications to users based on reminders.
- **S3**: Hosts the static frontend of the application.

## Steps to Implement

### Step 1: Configure Simple Email Service (SES)

Configure AWS SES to send emails. Set up email addresses for the application to use for sending reminders, and verify email addresses to comply with SES sandbox restrictions (if applicable).

### Step 2: Add an Email Lambda Function

Create a Lambda function that integrates with AWS SES to send email notifications. This Lambda will be triggered to send an email when a reminder needs to be processed.

```python
import boto3, os, json

FROM_EMAIL_ADDRESS = 'REPLACE_ME'

ses = boto3.client('ses')

def lambda_handler(event, context):
    # Print event data to logs .. 
    print("Received event: " + json.dumps(event))
    # Publish message directly to email, provided by EmailOnly or EmailPar TASK
    ses.send_email( Source=FROM_EMAIL_ADDRESS,
        Destination={ 'ToAddresses': [ event['Input']['email'] ] }, 
        Message={ 'Subject': {'Data': 'Whiskers Commands You to attend!'},
            'Body': {'Text': {'Data': event['Input']['message']}}
        }
    )
    return 'Success!'
  
```

### Step 3: Implement and Configure the State Machine

Create an AWS Step Functions state machine, which will be the core of the reminder processing logic. This state machine will trigger actions like sending reminder emails via the Lambda function and waiting based on user-defined intervals.

```json
{
  "Comment": "Using Lambda for email.",
  "StartAt": "Timer",
  "States": {
    "Timer": {
      "Type": "Wait",
      "SecondsPath": "$.waitSeconds",
      "Next": "Email"
    },
    "Email": {
      "Type" : "Task",
      "Resource": "arn:aws:states:::lambda:invoke",
      "Parameters": {
        "FunctionName": "EMAIL_LAMBDA_ARN",
        "Payload": {
          "Input.$": "$"
        }
      },
      "Next": "NextState"
    },
    "NextState": {
      "Type": "Pass",
      "End": true
    }
  }
}
```

### Step 4: Implement the API Gateway, API, and Supporting Lambda Function

Set up an API Gateway to provide endpoints for interacting with the application. Implement the backend logic via a Lambda function that processes requests and communicates with the state machine.

API lambda code: 

```json

# This code is a bit ...messy and includes some workarounds
# It functions fine, but needs some cleanup
# Checked the DecimalEncoder and Checks workarounds 20200402 and no progression towards fix

import boto3, json, os, decimal

SM_ARN = 'YOUR_STATEMACHINE_ARN'

sm = boto3.client('stepfunctions')

def lambda_handler(event, context):
    # Print event data to logs .. 
    print("Received event: " + json.dumps(event))

    # Load data coming from APIGateway
    data = json.loads(event['body'])
    data['waitSeconds'] = int(data['waitSeconds'])
    
    # Sanity check that all of the parameters we need have come through from API gateway
    # Mixture of optional and mandatory ones
    checks = []
    checks.append('waitSeconds' in data)
    checks.append(type(data['waitSeconds']) == int)
    checks.append('message' in data)

    # if any checks fail, return error to API Gateway to return to client
    if False in checks:
        response = {
            "statusCode": 400,
            "headers": {"Access-Control-Allow-Origin":"*"},
            "body": json.dumps( { "Status": "Success", "Reason": "Input failed validation" }, cls=DecimalEncoder )
        }
    # If none, start the state machine execution and inform client of 2XX success :)
    else: 
        sm.start_execution( stateMachineArn=SM_ARN, input=json.dumps(data, cls=DecimalEncoder) )
        response = {
            "statusCode": 200,
            "headers": {"Access-Control-Allow-Origin":"*"},
            "body": json.dumps( {"Status": "Success"}, cls=DecimalEncoder )
        }
    return response

# This is a workaround for: http://bugs.python.org/issue16535
# Solution discussed on this thread https://stackoverflow.com/questions/11942364/typeerror-integer-is-not-json-serializable-when-serializing-json-in-python
# https://stackoverflow.com/questions/1960516/python-json-serialize-a-decimal-object
# Credit goes to the group :)
class DecimalEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, decimal.Decimal):
            return int(obj)
        return super(DecimalEncoder, self).default(obj)

```

### Step 5: Implement the Static Frontend and Test Functionality

Develop a simple static frontend (e.g., HTML/CSS/JavaScript) hosted on S3 to allow users to interact with the API. Test the end-to-end functionality by creating reminders, triggering email notifications, and verifying that the state machine works as expected.

Bucket Policy: 

```python
{
    "Version":"2012-10-17",
    "Statement":[
      {
        "Sid":"PublicRead",
        "Effect":"Allow",
        "Principal": "*",
        "Action":["s3:GetObject"],
        "Resource":["REPLACEME_PET_CUDDLE_O_TRON_BUCKET_ARN/*"]
      }
    ]
  }

```

### Step 6: Cleanup the Account

Once the application is deployed and verified, ensure you clean up any resources no longer needed to avoid unnecessary charges. This includes deleting Lambda functions, API Gateway configurations, and Step Functions state machines.

## Requirements

- AWS Account with access to SES, Lambda, API Gateway, Step Functions, and S3.
- Knowledge of Python for Lambda function development.
- Basic understanding of serverless architecture and AWS services.
