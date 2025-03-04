# EC2 Auto Scheduler

This guide explains how to create an **EC2 Auto Scheduler** to start and stop your EC2 instances at specified times, using **AWS Lambda** and **CloudWatch Events**.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Creating Lambda Functions](#creating-lambda-functions)
3. [Setting Up CloudWatch Events](#setting-up-cloudwatch-events)
4. [IAM Role for Lambda](#iam-role-for-lambda)
5. [Testing the Scheduler](#testing-the-scheduler)
6. [Conclusion](#conclusion)

---

## Prerequisites

Before starting, ensure you have the following AWS services set up:

- **AWS EC2 Instances**: The EC2 instances you want to schedule (ensure they are running).
- **AWS Lambda**: A serverless compute service for scheduling EC2 instance start/stop operations.
- **AWS CloudWatch**: To trigger the Lambda function based on a cron schedule.
- **IAM Role**: Ensure you have appropriate permissions to start and stop EC2 instances.

---

## Creating Lambda Functions

### 1. Create Lambda Function to Start EC2 Instances

1. **Go to AWS Lambda Console**: Navigate to [AWS Lambda](https://console.aws.amazon.com/lambda/).
2. **Create a new Lambda function**:
   - **Function name**: `start-ec2-instances`
   - **Runtime**: Python 3.8 or Node.js
   - **Role**: Select `Create a new role with basic Lambda permissions`.

3. **Add Lambda code** to start EC2 instances:

```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    
    # Specify your EC2 instance IDs here
    instance_ids = ['i-1234567890abcdef0', 'i-0987654321abcdef0']
    
    # Start EC2 instances
    ec2.start_instances(InstanceIds=instance_ids)
    
    print(f"Started EC2 instances: {instance_ids}")
    return {
        'statusCode': 200,
        'body': f"Started EC2 instances: {instance_ids}"
    }
```
4. Deploy the lambda function.
## 2. Create Lambda Function to Stop EC2 Instances
Follow the same steps as above to create another Lambda function:

- Function name: ```stop-ec2-instances```
Add Lambda code to stop EC2 instances:
```python
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')
    
    # Specify your EC2 instance IDs here
    instance_ids = ['i-1234567890abcdef0', 'i-0987654321abcdef0']
    
    # Stop EC2 instances
    ec2.stop_instances(InstanceIds=instance_ids)
    
    print(f"Stopped EC2 instances: {instance_ids}")
    return {
        'statusCode': 200,
        'body': f"Stopped EC2 instances: {instance_ids}"
    }
```
Deploy the Lambda function.
## Setting Up CloudWatch Events
### Create CloudWatch Rule to Trigger EC2 Start Function
- Go to the AWS CloudWatch Console.
- Create a new rule:
Event Source: ```Schedule```
**Cron expression:** Specify the schedule to start EC2 instances ```(e.g., cron(0 8 * * ? *)``` to start at 8 AM UTC every day).
**Target:** Select the Lambda function ```(start-ec2-instances)```
**Configure permissions:** AWS will prompt you to create the necessary permissions for CloudWatch to invoke your Lambda function. Grant the permissions.
- Save the rule.

### Create CloudWatch Rule to Trigger EC2 Stop Function
Repeat the steps above to create a rule to stop the EC2 instances.
- **Event Source:** Schedule
- **Cron expression:** Specify the schedule to stop EC2 instances (e.g., cron(0 18 * * ? *) to stop at 6 PM UTC every day).
- **Target:** Select the Lambda function (stop-ec2-instances).
Configure permissions and save the rule.
## IAM Role for Lambda
Make sure the Lambda function has the proper permissions to start and stop EC2 instances. Create an IAM role with the following policy:

### IAM Policy to Allow EC2 Start/Stop
1. Go to the IAM Console.
2. Create a new policy with the following JSON:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "*"
        }
    ]
}
```
3. Attach this policy to the Lambda execution role.
## Testing the Scheduler

Once you've set up the EC2 Auto Scheduler, it's important to verify that the Lambda functions and CloudWatch events are working as expected. Follow the steps below to test the functionality.

---

### 1. Test Lambda Functions

To ensure that the Lambda functions can start and stop the EC2 instances correctly, you can manually invoke the Lambda functions from the AWS Lambda Console.

#### Steps to Test Lambda Functions:

1. **Go to AWS Lambda Console**: Navigate to the [AWS Lambda Console](https://console.aws.amazon.com/lambda/).
2. **Select your Lambda function**: Choose the `start-ec2-instances` or `stop-ec2-instances` Lambda function.
3. **Click on "Test"**: In the top-right corner, click on the "Test" button.
4. **Configure a test event**: For a basic test, you can use the default test event configuration (no changes required).
5. **Click "Test" again**: This will trigger the Lambda function and start or stop the EC2 instances.
6. **Check the logs**: After the test completes, check the execution results in the "Logs" section to confirm the EC2 instances were started or stopped successfully.

---

### 2. Verify the Scheduled Events

CloudWatch Events trigger Lambda functions based on the defined schedule. You need to verify that the CloudWatch events are triggered correctly at the specified times.

#### Steps to Verify CloudWatch Events:

1. **Go to CloudWatch Console**: Navigate to the [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/).
2. **Select "Logs" from the menu**: In the left sidebar, click on the "Logs" section.
3. **Check the log group**: Look for the log group related to your Lambda functions (e.g., `/aws/lambda/start-ec2-instances` or `/aws/lambda/stop-ec2-instances`).
4. **Check the log stream**: Click on the log stream and check for logs at the scheduled times. The logs should indicate that the Lambda function was triggered and that it performed the EC2 start or stop operation.

---

### 3. Verify EC2 Instance States

To confirm that the EC2 instances are starting and stopping according to the defined schedule, follow these steps:

#### Steps to Verify EC2 Instance States:

1. **Go to EC2 Console**: Navigate to the [EC2 Console](https://console.aws.amazon.com/ec2/).
2. **Check Instance States**: Under the "Instances" section, look for your EC2 instances.
3. **Verify Instance State**: Confirm that the EC2 instances are starting and stopping at the correct times as per the CloudWatch schedule. If the schedule is set to start at 8 AM, ensure the instance is running at that time. Similarly, ensure the instance is stopped at the scheduled stop time (e.g., 6 PM).

---

By performing these tests, you can verify that the Lambda functions are properly triggering, the CloudWatch events are scheduled correctly, and the EC2 instances are starting and stopping as expected.
## Conclusion

You have successfully created an EC2 Auto Scheduler using **AWS Lambda** and **CloudWatch Events**. This setup allows you to automatically start and stop EC2 instances based on a predefined schedule, helping you save on costs during off-hours.

### Possible Improvements:

- **Fine-grained control over instance states**: You can extend the solution to provide more control over instance states, such as scheduling specific instances or groups of instances.
- **Integration with AWS Systems Manager**: You could integrate this setup with AWS Systems Manager to start/stop EC2 instances based on more advanced criteria, such as instance tags, instance types, or other metadata.

By following this setup, you can automate EC2 instance management and optimize your cloud infrastructure costs efficiently.
