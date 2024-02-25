<pre>
  SSM is an AWS software provisioning, configuration management service. SSM is an agent-based service. SSM also supports hybrid infrastructure. SSM used to remotely execute the tasks, connecting remote servers, patch management, inventory management, Consistent Configuration, etc.

In our infrastructure, we want to install and configure security and monitoring software on all existing and newly launched instances automatically. One way was building AMI with all required software, but it will create issues in patching and configuration update. Every time we have to update AMI for all instances. To avoid this we choose SSM to install and configure the software.

We decided to keep all our installation/configuration scripts on AWS S3 and from S3 SSM will install to all selected EC2 instances. But AWS SSM supports only https:// S3 URL means the bucket should be public with limited permission. SSM does not s3:// URL. See the below SSM CLI example command.

aws ssm send-command \
    --document-name "AWS-RunRemoteScript" \
    --targets "Key=instanceids,Values=i-02573cafcfEXAMPLE" \
    --parameters '{"sourceType":["S3"],"sourceInfo":["{\"path\":\"https://s3.amazonaws.com/doc-example-bucket/scripts/shell/helloWorld.sh\"}"],"commandLine":["helloWorld.sh argument-1 argument-2"]}'
Keeping files in public S3 is not advisable, it may contain some sensitive data like licenses, passwords, etc.

We solved this issue.

let us get started !!!

Note: We implemented this for AWS ec2 instances, not tested it for hybrid infra.

Prerequisites for this are

a. SSM agent must be installed and configured on all AWS EC2.

https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-manual-agent-install.html

b. AWS CLI on all EC2 instances.

We will use the following AWS resources.

AWS S3
AWS IAM Policy
SSM Custom Document
SSM State Manger
Step 1: AWS private S3 bucket:

a. Create an AWS S3 bucket. By default every S3 bucket is private. Lets us create an S3 named “ssm-demo”


AWS S3 Service

b. Create a shell script named “ssm-demo.sh” to install and configure the software.

#!/bin/bash
echo "Installing Ngnix"
sudo yum install nginx
sudo /etc/init.d/nginx start
c. Upload ssm-demo.sh to S3 bucket.

Step 2. AWS IAM Policy: Create an IAM policy named “ssm-demo-policy” with reading permission to the S3 bucket that contains all scripts.

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::ssm-demo/*"
        }
    ]
}
Step 3. Attach Policy to IAM Instance Profile: Each SSM managed EC2 instance attached with one or more IAM instance profiles. Attach the above policy to one of the IAM instance profiles.

Step 4. SSM Custom Document: An AWS Systems Manager document (SSM document) defines the actions that Systems Manager performs on your managed instances. Systems Manager includes more than 100 pre-configured documents. We can also create custom SSM documents. Below we will create a custom document. In this document, we do the following actions.

a. Copy installation script to EC2 instances /opt dircctory.

sudo aws s3 cp s3://ssm-demo/ssm-demo.sh /opt

b. Make ssm-demo.sh to executable

sudo chmod 755 /opt/ssm-demo.sh

c. Execute the shell script.

cd /opt/ && ./ssm-demo.sh

Go to the AWS SSM Console.


AWS SSM Console
Choose documents from SSM left menu panel.


SSM Menu Panel
Choose to Create document → Command and Session


Filled in the details like the name of the document, target type “/AWS::EC2::Instance", Document type “command document” and in content write your scripts like below.

{
  "schemaVersion": "2.2",
  "description": "Command Document For Installing Agents",
  "parameters": {},
  "mainSteps": [
    {
      "action": "aws:runShellScript",
      "name": "example",
      "inputs": {
        "runCommand": [
          "sudo aws s3 cp s3://ssm-demo/ssm-demo.sh /opt",
          "sudo chmod 755 /opt/ssm-demo.sh",
          "cd /opt/ && ./ssm-demo.sh"
        ]
      }
    }
  ]
}
Then “Create document”.


Step. 4 SSM State Manger: State Manager, a capability of AWS Systems Manager, is a secure and scalable configuration management service.

Create an association with the custom document.

a. Go to SSM state manager console


Click on State Manager
b. Choose “Create An Association”

c. Fill in the Name of your association

d. Select the custom document by applying the filter “Owner: Owned by me” and choose the document version as well.


e. Choose the target instances based on given options like based on tags, manual choice, etc.


f. We can schedule the association as per the requirement.


g. We can also configure s3 to write association execution output.


Then create the Association.
</pre>
