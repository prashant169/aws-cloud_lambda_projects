# lambda_ebs_volume_modification

PROJECT DESCRIPTION --------------------------------------- >>
As a Cloud Engineering team we take care of the AWS environment and make sure it is in compliance with the organizational policies. We use AWS cloud watch in combination with AWS Lambda to govern the resources according to the policies. For example, we Trigger a Lambda function when an Amazon Elastic Block Store (EBS) volume is created. We use Amazon CloudWatch Events. CloudWatch Events that allows us to monitor and respond to EBS volumes that are of type GP2 and convert them to type GP3.-------------------**


![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/3e2d1ffe-b6b2-4c39-b1cc-a78e5fbe8c19)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/6fcf6312-4268-41e3-863b-2424afbf029b)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/d24ccd9a-ad3f-4865-a3bc-4bc7a377efcf)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/f791b80d-3383-4e5b-a5f6-5ab406c3cf4e)

we can see the basic python lambda function: 

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/11c623bd-8d78-422d-9e1f-e5fc2435b281)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/b8fd9432-937d-499f-baa0-a456e8dd55d2)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/0f07da45-dfde-40d3-a6a9-dd6435c9747c)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/740ba9ba-ed7f-41ca-957d-de1409253e8b)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/9e440960-5816-4593-88c5-d07fa6cd9ba1)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/f72bb925-0dd2-4852-81f9-1725b1f69086)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/40815ed8-d0cb-453c-8298-e7b5d0915702)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/9a2e1acd-a4ce-48bd-930b-3d716ea624c2)

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/6f3c0a9c-ac46-42cd-9e7e-663586accc17)


 Now, edit the lambda functionadd print(event)..i.e it is importing event in json from cloudwatch logs. So to verify the event we are using print the event.-->test is from the cloudwatch logs by creating a new volume

 ![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/5d9d1178-03a1-4280-abd3-dec407cb52ce)

Now, we can notify the complete details of the EBS volume which is in the json data:-

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/f8f9821e-1939-4462-bc1c-a4bdc1cb9a91)

Appendix:

-copy this Json as marked above and investigate it thru online Json formator and validator

![image](https://github.com/prashant169/aws-cloud_lambda_projects/assets/78464585/edf86b41-7499-4a99-9aa3-616d607e16cc)

 {
 
   "version":"0",
 
   "id":"ca699fb7-08ef-ef50-9209-b9b42de37008",
   
   "detail-type":"EBS Volume Notification",
   
   "source":"aws.ec2",
   
   "account":"307620414606",
   
   "time":"2023-09-19T09:47:51Z",
   
   "region":"ap-south-1",
   
   "resources":[
   
      "arn:aws:ec2:ap-south-1:307620414606:volume/vol-020b7e5c69dd9566b"
   
   ],

      "detail":{
     
      "result":"available",
      
      "cause":"",
     
      "event":"createVolume",
     
      "request-id":"0b92e70a-c6f0-4cb2-a474-ebe1795aefe0"
      
   }

}

i.e., This lambda function translated event into Json data (event=Json data)
-now I need the ID of my volume from this event, to convert into gp3(by importing python module boto3 package/functionality into our function) as per our project. So, we have to extract the volume id out of entire Json data

Import boto3

Def get_volume_id_from_arn(volume_arn):

(i.e., here we are doing parsing:- Parsing JSON means converting JSON (JavaScript Object Notation) data into a format that can be easily used and manipulated in a programming language 

i.e., volume_arn= event [resources][0] (It means from the json event resources first entry-as 0)we are giving to our lambda def under get_volume_id

-similarly we are getting volume-id from parsing as mentioned below:

volume_id= get_volume_id_from_arn(volume_arn)

-now go to boto3 modify_volume documentation for syntax to convert our gp2 volume id into gp3

-as we know ebs falls under ec2, so I am calling the ec2_client=boto3.client(‘ec2’)

So if we put all together, the final code to deploy in lambda as:-


import boto3

def get_volume_id_from_arn(volume_arn):
   
    # Split the ARN using the colon(':') separator
    
    arn_parts = volume_arn.split(':')
    
    # The volume ID is the last part of the ARN after the 'volume/'  prefix
    
    volume_id = arn_parts[-1].split('/')[-1]
    
    return volume_id
    
    
def lambda_handler(event, context):
    
    volume_arn = event['resources'][0]
    volume_id = get_volume_id_from_arn(volume_arn)
    
    ec2_client = boto3.client('ec2')
    
    response = ec2_client.modify_volume(
        VolumeId=volume_id,
        VolumeType='gp3',
    )
    

Note: create any gp2 volume and check whether it is automatically converted into gp3.


















