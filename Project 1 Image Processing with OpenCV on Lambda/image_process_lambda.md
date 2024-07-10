Title: Image Processing with AWS Lambda

•	Introduction:

•	Overview of the project.

•	Goals: Convert uploaded images to black and white, store in S3, and email results.

To accomplish the described task, you can follow these general steps using AWS services:

Architecture Overview

Title: System Architecture

•	Components:

•	S3 Buckets (Source)

•	Lambda Function

•	SES (Simple Email Service)

•	Flow:

•	Image upload triggers Lambda.

•	Lambda processes the image and stores it in the Destination S3 bucket.

•	Email sent with original and processed images.

AWS Resources Setup

Title: Setting Up AWS Resources

•	Tasks:

•	Create two S3 buckets.

•	Set up an IAM role for Lambda.

•	Configure SNS for sending emails.


1.0 Amazon S3:
•	Create two S3 buckets, one for the original images and one for the processed (black/white) images.

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/7f07eedf-b248-455f-a3f6-cf0f45b1b8c4)


Lambda Function

Title: Lambda Function Configuration

•	Configuration:

•	Create a Lambda function.

•	Set up the IAM role with necessary permissions.

•	Configure S3 trigger event.

•	Code:

•	Python code using OpenCV for image processing.

2.0 AWS Lambda:

•	Create an AWS Lambda function that triggers on an S3 bucket event (e.g., object creation).

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/5dc3c1dc-9e11-40d4-8098-0dad61c71b96)

•	Configure the Lambda function to use the appropriate IAM role with permission to read from the source S3 bucket, write to the destination S3 bucket, and send emails.

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/866302c2-f777-4756-9038-2e8b2c89bce3)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/553881f4-52ae-482b-86f2-cecc47294873)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/8d25272d-048f-438d-bd1c-7cfb7079e4cc)

In Lambda-Add Trigger:

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/c0f3b70c-bfeb-41c3-9f52-8b9381c8a6c6)

3.0 Python with OpenCV:
•	Write a Lambda function in Python that uses the OpenCV library to convert the image to black/white.
•	Ensure that the necessary OpenCV dependencies are included in the deployment package.
OpenCV is a huge open-source library for computer vision, machine learning, and image processing.
OpenCV supports a wide variety of programming languages like Python, C++, Java, etc. It can process images and videos to identify objects, faces, or even the handwriting of a human.

Certainly! OpenCV (Open Source Computer Vision Library) is a powerful tool for computer vision, image processing, and machine learning tasks. It provides a wide range of functions and algorithms to work with images, videos, and other visual data. Here are some details about using OpenCV in Python:
Installation:

-To install OpenCV in Python, you can use pip, the package manager for Python. Depending on your operating system, execute one of the following commands:
	Windows: pip install opencv-python

	macOS: brew install opencv3 --with-contrib --with-python3

	Linux: sudo apt-get install libopencv-dev python-opencv

Importing Package into our Lambda Layers:-

 Steps:
 
1.	Connect to UBUNTU VM from EC2 and run the following commands to install required packages for the project.

sudo apt-get update -y

python3 --version

sudo apt install python3-pip

sudo apt install awscli

mkdir -p build/python/lib/python3.8/site-packages

pip3 install opencv-python-headless -t build/python/lib/python3.8/site-packages

cd build

sudo apt install zip

zip -r package.zip .

ls (#we can notice our zip file)

aws s3 cp package.zip bucket_name_in_s3 

(#this command will help u to copy the zip file into s3 bucket u have created, in my case I have created OpenCV-layers-project)


![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/57a40b77-9e43-455d-b496-8ef0b32c955c)


Lambda Code Deployment

Title: Deploying Lambda Function

•	Deployment:

•	Create a layer in our lambda function and upload our zip file from S3 bucket. i.e. layers help the lambda function to import the packages required to execute our code.

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/e640ed3a-de10-4fe5-8485-696eab90bc52)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/11f3e547-21e5-4e1d-ae22-2314baf4bfd8)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/272d8145-aec0-4272-84d3-d5053664ffa9)

Now, upload this code in the lambda function:-

import os
import cv2
import boto3
import numpy as np

s3 = boto3.client('s3')

def lambda_handler(event, context):
      # Get the bucket and key from the S3 event
       source_bucket = event['Records'][0]['s3']['bucket']['name']
source_key = event['Records'][0]['s3']['object']['key']
    
    # Read the image from the source S3 bucket
    response = s3.get_object(Bucket=source_bucket, Key=source_key)
    image_content = response['Body'].read()
    nparr = np.frombuffer(image_content, np.uint8)
    img = cv2.imdecode(nparr, cv2.IMREAD_COLOR)

    # Convert image to black and white
    gray_img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

    # Encode black and white image to bytes
    _, bw_image_bytes = cv2.imencode('.jpg', gray_img)
    
    # Destination bucket where the black and white image will be uploaded
    destination_bucket = "guda-secondary"
    
    # Upload the black and white image to the destination S3 bucket
    destination_key = 'black_and_white_' + os.path.basename(source_key)
    s3.put_object(Body=bw_image_bytes.tobytes(), Bucket=destination_bucket, Key=destination_key)
    
    return {
        'statusCode': 200,
        'body': f'Image converted to black and white and saved as {destination_key}'


Now, deploy the file, so it will save your code.
If you are get any errors, kindly edit the general configuration of lambda under configuration tab, increase the timeout seconds.


![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/304f4318-a9b4-4e98-9143-86af8b2a9422)


Now, upload a image in your primary bucket, in my case it is (guda-primary). 
This will automatically trigger our lambda function to convert our image into black&white and uploaded back into our destination bucket as we mentioned in our code(guda-secondary).

My primary bucket with uploaded images…

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/75facda7-6a8e-4741-81ab-12a15652e414)

My secondary bucket with black and white conversion by lambda function….

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/6fe571f3-e91e-4783-a787-0ce043b7743d)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/527aa421-1aab-4ac7-a3f1-176ed1fc6f69)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/ec521128-ce02-40e6-826b-13032daa0e8c)






