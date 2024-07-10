#[Step-by-Step] How to start & stop AWS EC2 automatically using Lambda & EventBridge

Introduction
In this tutorial, we are going to automate starting and stopping EC2 via Lambda & EventBridge at regular intervals (e.g.running EC2 from 8:00 am to 8:00 pm). While the EC2 is stopped, we can save the compute cost during this time.
In order to let Lambda be able to start and stop the EC2 instance, we first need to assign the required IAM permission for lambda in Step1.
Once it is done, we can create two lambda functions, which the first one is for starting the EC2 and the second one is for stopping the EC2. After that, we can create the EventBridge Schedule to trigger the lambda function at the desired times.-------------------------------------------*

Step Summary:

Step 1: Create IAM role for Lambda

Step 2: Create 2 Lambda function for starting and stopping EC2

Step 3: Create 2 EventBridge Schedule to invoke 2 Lambda functions

Step Details:

Step 1: Create IAM role for Lambda: 

Go to IAM

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/a7aead3a-1def-4fe5-90a2-9b1d7ba9b68d)

Go to Role

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/7cffad1e-25f7-4bee-bd53-a613bc965b3f)

Press Create Role

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/499bfe05-2e1e-4c4a-9c68-d97d6eda3ded)

Select AWS Service and Lambda and press Next

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/10f17f94-d883-497f-81df-87172b5a615b)

Press Create Policy

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/b0b8710b-0519-45a0-a375-6ea4c7d6850c)

Press JSON

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/b679b196-d70e-489b-8830-5a9b3e4c7f58)

Copy the following IAM Policy and paste in JSON Editor

{
 
  "Version": "2012-10-17",
  
  "Statement": [
  
    {
    
      "Effect": "Allow",  
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Start*",
        "ec2:Stop*"
      ],
      "Resource": "*"
    }, 
    {
      "Effect": "Allow",
      "Action": [
          "ses:SendEmail",
          "ses:SendRawEmail"
      ],
      "Resource": "*"
    }
  ]
}


Go Back to Visual and Press Next

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/954292e2-c145-4dee-ac73-812dd632d7db)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/cb4fa53c-2a5f-4dd9-8edf-a39e56dd807e)

Name the IAM Policy and Press Create Policy

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/cb7d6042-d00c-4ed1-bcf4-a685a9b44dc6)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/8babcc51-6995-4c3d-b668-317f8ccbb38b)


Select the created IAM Policy and Press Next

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/29e7f093-4deb-4e62-906b-6f5e7e3ac455)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/9380b7bf-22d7-41aa-8375-2db039348561)

Name the IAM Role and Press Create Role

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/fb4db7e1-7ccd-4f50-a219-552bb5492032)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/275736ed-516a-44fb-bfbc-7e9e95b55697)

Now, you have created the IAM Role for Lambda

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/896d37ae-cd80-4d5c-b3fe-5dd05e25a339)

Step 2: Create 2 Lambda function for starting and stopping EC2

Go to Lambda

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/70218254-c827-4353-a0fd-f009735f28bf)

Press Create Function

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/921e0be6-0032-4c8f-8829-bb092714fd0c)

Select Author from scratch and Name the Lamdba function

Select Python 3.9 runtime and Select IAM Role for Lambda

Press Create function

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/f7d75e5f-14b0-4a96-b354-584e4f8c6ffe)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/0564cdf7-f90b-4fea-a2d4-6e6dde62215f)

Modify the following Python Code and paste in Code source Editor

AutoStartEC2

Replace 'ap-east-1' to the designated Region for your situation
Replace 'i-0f7f94a56f47e7221' to the EC2 ID
you can add more instances in this array if you want
E.g. ['i-0f7f94a56f47e7221','i-0uweq7w7d7293','i-0wwq78e78we781']
import boto3
/////////////////////////////////////////////////////
region = 'ap-east-1'
instances = ['i-0f7f94a56f47e7221']
/////////////////////////////////////////////////////
ec2 = boto3.client('ec2', region_name=region)
def lambda_handler(event, context):
    ec2.start_instances(InstanceIds=instances)
    print('started your instances: ' + str(instances))

Press Deploy


![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/8e5eaa41-fc94-4583-a8d9-64997d4803d8)

Now, you have created the lambda function for starting EC2

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/20dbecf8-8de8-40d6-939e-92ac3e952b03)

Repeat the above steps and refer to the following code to create the lambda function for stopping EC2

AutoStopEC2

import boto3
/////////////////////////////////////////////////////
region = 'ap-east-1'
instances = ['i-0f7f94a56f47e7221']
/////////////////////////////////////////////////////
ec2 = boto3.client('ec2', region_name=region)
def lambda_handler(event, context):
    ec2.stop_instances(InstanceIds=instances)
    print('stopped your instances: ' + str(instances))



![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/8833ad90-d79f-426b-bb91-c581f737ce60)


Step 3: Create 2 EventBridge Schedule to invoke 2 Lambda functions

Go to EventBridge

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/b80ca5b1-c014-4142-b50c-83daffdf000b)

Select EventBridge Schedule and Press Create schedule

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/4c093caf-fc57-411d-9e91-c01a5d93dd40)

Name the EventBridge Schedule

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/b9d4c51c-41fd-410e-8ca4-88bc997e2183)

Select Recurring schedule and Cron-based schedule

Define the cron expression for the schedule

cron(minutes hours day-of-month month day-of-week year)

For our cron expression, it will run at 8:00 am from Monday to Firday every week every year


Select Off in Flexible time window

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/f80c3a4a-f5d1-43cd-8765-badb535efe5e)

Select the Timezone and Press Next

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/71289675-5a4a-4838-8a51-bc53d45a16b1)

Select Lambda

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/bc84b835-6f7f-4b17-becf-8ad23d894bc6)

Select the Lambda Function and Press Next

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/56435214-1db4-46d7-9026-e44093a7e419)

Press Next

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/ced62f9c-e203-4646-94a4-0719ef24fabe)


![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/47332887-3324-443e-97a3-686430822c8d)

Press Create schedule

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/ff1845ea-596a-47c6-a786-b7c7f0986fa9)

Now, you have created the EventBridge Schedule for triggering lambda to start EC2
![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/3fc65906-ee86-46d8-ad68-872e73e289c7)

Repeat the above steps and Change the cron expression from 8 to 20 and select the AutoStopEC2 lambda function to create EventBridge Schedule for triggering lambda to stop EC2

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/2b1ed518-5d52-4522-8d75-b85a4e09b8a1)





































