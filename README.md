# Serverless-Web-APP

This project is a serverless reminder application built using AWS services, including Lambda, Step Functions, API Gateway, Simple Email Service (SES), and a static frontend. The application is designed to automate email reminders and provide users with real-time interactions via an API and frontend interface.

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

### Step 3: Implement and Configure the State Machine

Create an AWS Step Functions state machine, which will be the core of the reminder processing logic. This state machine will trigger actions like sending reminder emails via the Lambda function and waiting based on user-defined intervals.

### Step 4: Implement the API Gateway, API, and Supporting Lambda Function

Set up an API Gateway to provide endpoints for interacting with the application. Implement the backend logic via a Lambda function that processes requests and communicates with the state machine.

### Step 5: Implement the Static Frontend and Test Functionality

Develop a simple static frontend (e.g., HTML/CSS/JavaScript) hosted on S3 to allow users to interact with the API. Test the end-to-end functionality by creating reminders, triggering email notifications, and verifying that the state machine works as expected.

### Step 6: Cleanup the Account

Once the application is deployed and verified, ensure you clean up any resources no longer needed to avoid unnecessary charges. This includes deleting Lambda functions, API Gateway configurations, and Step Functions state machines.

## Requirements

- AWS Account with access to SES, Lambda, API Gateway, Step Functions, and S3.
- Knowledge of Python for Lambda function development.
- Basic understanding of serverless architecture and AWS services.
