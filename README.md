# WEEK 6: AWS VPC USING BASTION SERVER AND NAT GATEWAY PRACTICAL RECORD

## Aim:

To design and implement a secure AWS Virtual Private Cloud (VPC) architecture using a Bastion Server for secure administrative SSH access to private subnet resources and a NAT Gateway to provide outbound internet access for private instances.

---

# ARCHITECTURE OVERVIEW:

## VPC CIDR:

```text
10.0.0.0/16
```

## Public Subnet:

```text
10.0.1.0/24
```

Used for:
✔ Bastion Server
✔ NAT Gateway

## Private Subnet:

```text
10.0.2.0/24
```

Used for:
✔ Database Server

---

# PART A: LOGIN TO AWS CONSOLE

## Step 1: Login

1. Open browser.
2. Login to AWS Management Console.
3. Select preferred region:

   * Recommended: us-east-1

---

# PART B: CREATE CUSTOM VPC

## Step 2: Open VPC Dashboard

1. Search:

   ```
   VPC
   ```
2. Select:
   **VPC Dashboard**

---

## Step 3: Create VPC

1. Click:
   **Create VPC**
2. Choose:
   **VPC only**

### Enter:

### Name:

```text
Week6VPC
```

### IPv4 CIDR:

```text
10.0.0.0/16
```

### Click:

**Create VPC**

---

## Output:

✔ Custom VPC created successfully.

---

# PART C: CREATE SUBNETS

## Step 4: Create Public Subnet

1. VPC → Subnets → Create subnet

### Fill:

### Name:

```text
PublicSubnet
```

### Availability Zone:

```text
us-east-1a
```

### CIDR:

```text
10.0.1.0/24
```

### Click:

**Create subnet**

---

## Step 5: Create Private Subnet

1. Click:
   **Create subnet**

### Fill:

### Name:

```text
PrivateSubnet
```

### Availability Zone:

```text
us-east-1b
```

### CIDR:

```text
10.0.2.0/24
```

### Click:

**Create subnet**

---

## Output:

✔ Public and Private subnets created successfully.

---

# PART D: CREATE INTERNET GATEWAY

## Step 6: Create Internet Gateway

1. VPC → Internet Gateways
2. Click:
   **Create internet gateway**

### Name:

```text
Week6IGW
```

### Click:

**Create**

---

## Step 7: Attach Internet Gateway

1. Select gateway
2. Actions → Attach to VPC
3. Select:

```text
Week6VPC
```

4. Click:
   **Attach**

---

## Output:

✔ Public internet access enabled.

---

# PART E: CONFIGURE PUBLIC ROUTE TABLE

## Step 8: Create Public Route Table

1. VPC → Route Tables
2. Click:
   **Create route table**

### Name:

```text
PublicRouteTable
```

### VPC:

```text
Week6VPC
```

---

## Step 9: Add Internet Route

1. Routes → Edit routes
2. Add:

### Destination:

```text
0.0.0.0/0
```

### Target:

✔ Internet Gateway → Week6IGW

### Save changes

---

## Step 10: Associate Public Subnet

1. Subnet Associations → Edit
2. Select:
   ✔ PublicSubnet
3. Save

---

## Output:

✔ Public subnet configured successfully.

---

# PART F: CREATE SECURITY GROUPS

## Step 11: Bastion Security Group

1. EC2 → Security Groups → Create

### Name:

```text
BastionSG
```

### Inbound Rule:

✔ SSH (22)
✔ Source: My IP

---

## Step 12: Database Security Group

1. Create another security group

### Name:

```text
DatabaseSG
```

### Inbound Rules:

✔ SSH (22) from:

```text
10.0.1.0/24
```

✔ MySQL (3306) from:

```text
10.0.1.0/24
```

### Important:

❌ No public SSH access

---

## Output:

✔ Security architecture secured.

---

# PART G: CREATE KEY PAIRS

## Step 13:

1. EC2 → Key Pairs
2. Create:

### Bastion:

```text
bastion.pem
```

### Database:

```text
dbserver.pem
```

---

# PART H: LAUNCH BASTION SERVER

## Step 14: Launch EC2 Instance

### Name:

```text
BastionServer
```

### OS:

✔ Ubuntu

### VPC:

✔ Week6VPC

### Subnet:

✔ PublicSubnet

### Public IP:

✔ Enable

### Security Group:

✔ BastionSG

### Key Pair:

✔ bastion.pem

### Launch instance

---

## Output:

✔ Bastion server running.

---

# PART I: LAUNCH DATABASE SERVER

## Step 15: Launch Private EC2 Instance

### Name:

```text
DatabaseServer
```

### OS:

✔ Ubuntu

### VPC:

✔ Week6VPC

### Subnet:

✔ PrivateSubnet

### Public IP:

❌ Disabled

### Security Group:

✔ DatabaseSG

### Key Pair:

✔ dbserver.pem

### Launch instance

---

## Output:

✔ Private database server running.

---

# PART J: COPY DATABASE KEY TO BASTION SERVER

## Step 16: Local Machine Command

Run:

```bash
scp -i bastion.pem dbserver.pem ubuntu@<BastionPublicIP>:/home/ubuntu/
```

### Purpose:

Copies DB key securely to Bastion.

---

# PART K: CONNECT TO BASTION SERVER

## Step 17:

```bash
ssh -i bastion.pem ubuntu@<BastionPublicIP>
```

### Verify:

```bash
ls
```

### Expected:

✔ dbserver.pem visible

---

# PART L: CONNECT TO PRIVATE DATABASE SERVER

## Step 18:

```bash
chmod 400 dbserver.pem
ssh -i dbserver.pem ubuntu@10.0.2.x
```

### Replace:

`10.0.2.x` with private IP of DB server.

---

## Output:

✔ Successful secure SSH into private server.

---

# PART M: VERIFY PRIVATE SUBNET INTERNET RESTRICTION

## Step 19:

Run:

```bash
sudo apt update
```

### Expected:

❌ Fails due to no internet access

---

# PART N: CREATE NAT GATEWAY

## Step 20: Open NAT Gateway

1. VPC → NAT Gateways
2. Click:
   **Create NAT Gateway**

### Name:

```text
Week6NAT
```

### Subnet:

✔ PublicSubnet

### Elastic IP:

✔ Allocate Elastic IP

### Click:

**Create NAT Gateway**

---

## Output:

✔ NAT Gateway available.

---

# PART O: CREATE PRIVATE ROUTE TABLE

## Step 21: Create Route Table

1. VPC → Route Tables
2. Click:
   **Create route table**

### Name:

```text
PrivateRouteTable
```

### VPC:

✔ Week6VPC

---

## Step 22: Add NAT Route

### Destination:

```text
0.0.0.0/0
```

### Target:

✔ NAT Gateway

---

## Step 23: Associate Private Subnet

1. Subnet associations
2. Select:
   ✔ PrivateSubnet

---

## Output:

✔ Private subnet now has outbound internet.

---

# PART P: VERIFY NAT INTERNET ACCESS

## Step 24: Reconnect to Private Server

Repeat Bastion → DB connection.

---

## Step 25:

Run:

```bash
sudo apt update
```

### Expected:

✔ Internet works successfully
✔ Packages update successfully

---

# PART Q: OPTIONAL ADMIN TASKS

## Install packages:

```bash
sudo apt install mysql-server -y
```

OR

```bash
sudo apt install nginx -y
```

---

## Restart service:

```bash
sudo systemctl restart mysql
```

---

# PART R: REQUIRED SUBMISSION PROOF

## Recommended Screenshots:

✔ VPC architecture
✔ Public & Private subnets
✔ Bastion server launch
✔ DB server launch
✔ SCP key copy
✔ Bastion login
✔ Private DB login
✔ apt update failure before NAT
✔ NAT Gateway creation
✔ Route table configuration
✔ apt update success after NAT

---

# PART S: CLEANUP

## Delete:

✔ EC2 instances
✔ NAT Gateway
✔ Elastic IP
✔ Route tables
✔ Internet Gateway
✔ Security Groups
✔ Subnets
✔ VPC
✔ Key pairs

---

# FINAL OUTPUT:

✔ Secure VPC designed
✔ Bastion server access working
✔ Private server isolated
✔ NAT internet enabled
✔ Administrative operations successful

---

# Viva Questions:

## 1. What is a Bastion Server?

A public server used for secure administrative access to private instances.

## 2. Why is NAT Gateway required?

To allow private subnet outbound internet access.

## 3. Why is private server secure?

No public IP and restricted security groups.

## 4. Why does apt update fail initially?

No internet route exists for private subnet.

## 5. Why is NAT placed in public subnet?

It requires internet gateway connectivity.

## 6. Can Bastion replace NAT?

No. Bastion provides SSH only, NAT provides internet access.

---

# Final Conclusion:

The Week 6 practical successfully demonstrates enterprise-grade secure AWS networking using custom VPCs, subnet isolation, Bastion architecture, NAT Gateway, route management, and secure cloud administration.


===================================================================================================
=================================================================================================================


WEEK 7: S3 → LAMBDA → DYNAMODB INTEGRATION
Aim:

To create an automated serverless workflow where uploading a file to an S3 bucket triggers a Lambda function, which stores file metadata in a DynamoDB table.

ARCHITECTURE:

S3 (Upload) → Lambda (Trigger) → DynamoDB (Store data)

PART A: CREATE S3 BUCKET
Step 1: Login
Open AWS Console
Login
Select region (same for all services)
Step 2: Open S3

Search:

S3

Click Amazon S3

Step 3: Create Bucket
Click Create bucket
Fill:

Bucket name:

week7-s3-trigger-bucket-1234

✔ Enable Versioning
✔ Keep default settings

Click Create bucket
Output:

✔ S3 bucket created

PART B: CREATE DYNAMODB TABLE
Step 4: Open DynamoDB

Search:

DynamoDB
Step 5: Create Table
Click Create table
Fill:

Table name:

newtable

Partition key:

unique

Type: String

Click Create table
Output:

✔ DynamoDB table created

PART C: CREATE LAMBDA FUNCTION
Step 6: Open Lambda

Search:

Lambda
Step 7: Create Function
Click Create function
Choose:

✔ Author from scratch

Fill:

Function name:

S3ToDynamoDBFunction

Runtime:
✔ Python 3.x

Permissions:

✔ Create new role with basic Lambda permissions

Click:

Create function

Output:

✔ Lambda created

PART D: ADD LAMBDA CODE
Step 8: Add Code

Scroll to Code source

Replace code with:

import boto3
from uuid import uuid4

def lambda_handler(event, context):
    dynamodb = boto3.resource('dynamodb')

    if 'Records' in event:
        for record in event['Records']:
            bucket_name = record['s3']['bucket']['name']
            object_key = record['s3']['object']['key']
            size = record['s3']['object'].get('size', -1)
            event_name = record.get('eventName', 'Unknown')
            event_time = record.get('eventTime', 'Unknown')

            table = dynamodb.Table('newtable')

            table.put_item(
                Item={
                    'unique': str(uuid4()),
                    'Bucket': bucket_name,
                    'Object': object_key,
                    'Size': size,
                    'Event': event_name,
                    'EventTime': event_time
                }
            )
Step 9: Deploy

Click:
Deploy

Output:

✔ Lambda code deployed

PART E: ADD S3 TRIGGER
Step 10: Add Trigger
Go to Lambda function page
Click Add trigger
Configure:

Trigger:
✔ S3

Bucket:

week7-s3-trigger-bucket-1234

Event type:
✔ All object create events

✔ Check acknowledgment box

Click:
Add

Output:

✔ S3 connected to Lambda

PART F: ADD DYNAMODB PERMISSION
Step 11: Open IAM Role
Go to:
Lambda → Configuration → Permissions
Click execution role
Step 12: Add Policy
Click Add permissions → Create inline policy
Choose JSON

Paste:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "dynamodb:PutItem",
      "Resource": "arn:aws:dynamodb:*:*:table/newtable"
    }
  ]
}
Name:
lambda-dynamo-policy

Click:
Create policy

Output:

✔ Lambda can write to DynamoDB

PART G: TEST WORKFLOW
Step 13: Upload File to S3
Open S3 bucket
Click Upload
Choose any file
Click Upload
Output:

✔ File uploaded
✔ Lambda triggered automatically

PART H: VERIFY OUTPUT
Step 14: Check DynamoDB
Open DynamoDB
Select table:
newtable
Click Explore table items
Click Run
Expected:

✔ Record appears with:

Bucket name
File name
File size
Event type
Time
PART I: MONITOR LOGS
Step 15: CloudWatch Logs
Open Lambda
Click Monitor
Click View CloudWatch logs
Expected:

✔ Execution logs visible

PART J: OUTPUT

✔ S3 bucket created
✔ DynamoDB table created
✔ Lambda function created
✔ S3 trigger configured
✔ File upload successful
✔ Lambda executed
✔ DynamoDB updated
✔ Logs generated


=========================================================================================
===================================================================================================

# WEEK 8: AWS SNS, SQS, S3 → SNS → SQS → LAMBDA SERVERLESS EVENT-DRIVEN ARCHITECTURE PRACTICAL RECORD

## Aim:

To implement AWS messaging and event-driven cloud services by configuring Amazon SNS, Amazon SQS, Amazon S3 event notifications, and AWS Lambda for automated notifications, asynchronous processing, and serverless workflows.

---

# ARCHITECTURE OVERVIEW:

## Part A:

SNS Topic → Email Notification

## Part B:

S3 Upload → SNS → Email Notification

## Part C:

SQS Queue → Message Storage and Retrieval

## Part D:

S3 Upload → SNS → SQS → Lambda Processing

---

# PART A: SIMPLE NOTIFICATION SERVICE (SNS)

# Step 1: Login to AWS Console

1. Open browser.
2. Login to AWS account.
3. Select preferred region.

---

# Step 2: Open Amazon SNS

1. Search:

   ```
   SNS
   ```
2. Open:
   **Amazon Simple Notification Service**

---

# Step 3: Create SNS Topic

1. Left panel → Topics
2. Click:
   **Create topic**

### Configure:

### Type:

✔ Standard

### Name:

```text
MyEmailTopic
```

### Click:

**Create topic**

---

# Step 4: Create Email Subscription

1. Open created topic
2. Go to:
   **Subscriptions**
3. Click:
   **Create subscription**

### Configure:

### Protocol:

✔ Email

### Endpoint:

Enter your email address

### Click:

**Create subscription**

---

# Step 5: Confirm Subscription

1. Open email inbox
2. Find AWS SNS confirmation mail
3. Click:
   **Confirm subscription**

---

## Output:

✔ Email successfully subscribed

---

# Step 6: Publish Test Message

1. Open SNS topic
2. Click:
   **Publish message**

### Subject:

```text
Test Mail
```

### Message:

```text
Hello this is SNS test
```

### Click:

**Publish message**

---

## Output:

✔ Email notification received

---

# PART B: S3 → SNS → EMAIL NOTIFICATION

# Step 7: Create S3 Bucket

1. Search:

   ```
   S3
   ```
2. Open Amazon S3
3. Click:
   **Create bucket**

### Bucket name example:

```text
my-upload-bucket-2026
```

### Important:

✔ Same region as SNS topic

### Click:

**Create bucket**

---

# Step 8: Create / Reuse SNS Topic

### Example:

```text
S3UploadNotification
```

### Ensure:

✔ Email subscription confirmed

---

# Step 9: Add SNS Access Policy for S3

1. Open SNS Topic
2. Edit access policy
3. Add S3 publish permissions

### Purpose:

Allows S3 bucket to publish notifications to SNS.

---

# Step 10: Configure S3 Event Notification

1. Open S3 bucket
2. Go to:
   **Properties**
3. Scroll to:
   **Event notifications**
4. Click:
   **Create event notification**

### Configure:

### Event name:

```text
UploadNotification
```

### Event type:

✔ All object create events

### Destination:

✔ SNS Topic

### Select:

```text
S3UploadNotification
```

### Save changes

---

# Step 11: Test Upload

1. Upload any file into bucket

---

## Expected Flow:

S3 Upload → SNS Topic → Email Notification

---

## Output:

✔ Upload email received automatically

---

# PART C: SIMPLE QUEUE SERVICE (SQS)

# Step 12: Open SQS

1. Search:

   ```
   SQS
   ```
2. Open:
   **Amazon Simple Queue Service**

---

# Step 13: Create Queue

1. Click:
   **Create queue**

### Type:

✔ Standard

### Name:

```text
MyQueue
```

### Click:

**Create queue**

---

## Output:

✔ Queue created successfully

---

# Step 14: Send Message

1. Open MyQueue
2. Click:
   **Send and receive messages**

### Message body:

```text
Hello this is SQS test message
```

### Click:

**Send message**

---

# Step 15: Receive Message

1. Click:
   **Poll for messages**

---

## Output:

✔ Message appears successfully

---

# PART D: S3 → SNS → SQS → LAMBDA

# Step 16: Create S3 Bucket

### Name example:

```text
mys3eventbucket123
```

---

# Step 17: Create SNS Topic

### Name:

```text
MyS3SNSTopic
```

### Copy:

✔ Topic ARN

---

# Step 18: Create SQS Queue

### Name:

```text
MyS3Queue
```

### Copy:

✔ Queue ARN

---

# Step 19: Subscribe SQS to SNS

1. Open SNS topic
2. Click:
   **Create subscription**

### Protocol:

✔ Amazon SQS

### Endpoint:

✔ MyS3Queue

---

## Output:

✔ SNS connected to SQS

---

# Step 20: Add SQS Access Policy

1. Open MyS3Queue
2. Edit access policy

### Allow:

✔ SNS to send messages to SQS

---

# Step 21: Add SNS Access Policy for S3

### Allow:

✔ S3 bucket to publish to SNS topic

---

# Step 22: Configure S3 Event Notification

1. S3 Bucket → Properties → Event notifications

### Event name:

```text
S3UploadEvent
```

### Event type:

✔ All object create events

### Destination:

✔ SNS Topic → MyS3SNSTopic

---

# Step 23: Test Upload

1. Upload file to S3 bucket

---

## Expected Flow:

S3 → SNS → SQS

---

# Step 24: Verify SQS Message

1. Open MyS3Queue
2. Poll for messages

---

## Output:

✔ S3 event notification visible in queue

---

# PART E: CONFIGURE LAMBDA CONSUMER

# Step 25: Create Lambda Function

1. Search:

   ```
   Lambda
   ```
2. Click:
   **Create function**

### Choose:

✔ Author from scratch

### Function name:

```text
SQSConsumerFunction
```

### Runtime:

✔ Python 3.x

### Click:

**Create function**

---

# Step 26: Add SQS Trigger

1. Open Lambda function
2. Click:
   **Add trigger**

### Select:

✔ SQS

### Queue:

```text
MyS3Queue
```

### Click:

**Add**

---

# Step 27: Add Lambda Code

Replace code with:

```python
def lambda_handler(event, context):

    for record in event['Records']:
        print("Message received from SQS:")
        print(record['body'])

    return {
        'statusCode': 200,
        'body': 'Message processed successfully'
    }
```

---

# Step 28: Deploy Code

### Click:

**Deploy**

---

## Final Architecture:

User Upload → S3 → SNS → SQS → Lambda → CloudWatch Logs

---

# PART F: MONITORING

# Step 29: CloudWatch Logs

1. Open Lambda function
2. Monitor tab
3. View CloudWatch logs

---

## Verify:

✔ Message processing logs
✔ Successful execution
✔ Debugging information

---

# PART G: REQUIRED SUBMISSION PROOF

## Recommended Screenshots:

✔ SNS Topic creation
✔ Email subscription confirmation
✔ SNS email received
✔ S3 bucket creation
✔ S3 event notification
✔ SQS queue creation
✔ Message send/receive
✔ SNS-SQS subscription
✔ Lambda function creation
✔ Lambda trigger
✔ CloudWatch logs

---

# PART H: CLEANUP

## Delete:

✔ SNS topics
✔ Email subscriptions
✔ SQS queues
✔ Lambda functions
✔ S3 buckets
✔ Uploaded objects
✔ Access policies

---

# FINAL OUTPUT:

✔ SNS messaging configured
✔ Email notifications working
✔ S3 upload alerts working
✔ SQS queue functioning
✔ S3 → SNS → SQS workflow completed
✔ Lambda consumer processing successful
✔ Serverless architecture demonstrated

---

# Viva Questions:

## 1. What is SNS?

AWS publish-subscribe notification service.

## 2. What is SQS?

AWS managed message queue service.

## 3. Why use SNS + SQS together?

For reliable decoupled event distribution.

## 4. What is Lambda’s role?

Automatically processes queued messages.

## 5. Why use CloudWatch?

For monitoring and troubleshooting.

## 6. Why is SQS useful?

Stores messages safely during high traffic.

## 7. What architecture is demonstrated?

Fully serverless event-driven processing.

---

# Common Errors:

## 1. Email not received:

* Subscription not confirmed

## 2. S3 not publishing:

* SNS policy missing

## 3. SQS not receiving:

* Queue policy missing

## 4. Lambda not triggering:

* Trigger not configured

## 5. Duplicate messages:

* Message visibility timeout misconfiguration

---

# Final Conclusion:

The Week 8 practical successfully demonstrates AWS messaging services, event-driven architectures, decoupled cloud design, asynchronous processing, and serverless


==============================================================================================
======================================================================================================

# WEEK 9: IMPLEMENTATION OF ELASTIC LOAD BALANCER (ALB) WITH AUTO SCALING GROUP (ASG) USING AWS EC2 INSTANCES PRACTICAL RECORD

## Aim:

To deploy multiple EC2 web servers, configure an Application Load Balancer (ALB) to distribute incoming traffic, create an Auto Scaling Group (ASG) to automatically maintain and scale EC2 instances, and ensure high availability, fault tolerance, and performance under varying traffic loads.

---

# ARCHITECTURE OVERVIEW:

## Components:

✔ Two EC2 web servers running Apache
✔ Application Load Balancer (ALB)
✔ Target Group
✔ Launch Template
✔ Auto Scaling Group (ASG)
✔ CloudWatch metrics

---

# PART A: LAUNCH TWO EC2 WEB SERVERS

## Step 1: Launch EC2 Instance #1

1. Login to AWS Console.
2. Search:

   ```
   EC2
   ```
3. Open EC2 Dashboard.
4. Click:
   **Launch Instance**

### Configure:

### Name:

```text
webserver-1
```

### AMI:

✔ Amazon Linux 2

### Instance Type:

✔ t2.micro

### Security Group:

✔ SSH (22) from My IP
✔ HTTP (80) from Anywhere (0.0.0.0/0)

### Key Pair:

✔ Select/Create key pair

### Launch instance

---

## Step 2: Connect to webserver-1

### Run:

```bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "This is Server 1" > /var/www/html/index.html
```

---

## Output:

✔ Apache installed
✔ Web page configured

---

# Step 3: Launch EC2 Instance #2

### Repeat same process:

### Name:

```text
webserver-2
```

### Configure Apache:

```bash
yum update -y
yum install httpd -y
systemctl start httpd
systemctl enable httpd
echo "This is Server 2" > /var/www/html/index.html
```

---

## Output:

✔ Second web server configured

---

# PART B: CREATE LOAD BALANCER SECURITY GROUP

## Step 4: Create Security Group

1. EC2 → Security Groups → Create security group

### Name:

```text
lb-sg
```

### Inbound Rules:

✔ HTTP (80) from Anywhere
✔ Optional HTTPS (443)

---

## Output:

✔ Load balancer security group created

---

# PART C: CREATE APPLICATION LOAD BALANCER (ALB)

## Step 5: Open Load Balancers

1. EC2 → Load Balancers
2. Click:
   **Create Load Balancer**
3. Select:
   **Application Load Balancer**

---

## Step 6: Basic Configuration

### Name:

```text
my-alb
```

### Scheme:

✔ Internet-facing

### IP Type:

✔ IPv4

### Listener:

✔ HTTP (80)

---

## Step 7: Network Mapping

### VPC:

✔ Default VPC

### Availability Zones:

✔ Select all available subnets

---

## Step 8: Security Group

### Select:

```text
lb-sg
```

---

# PART D: CREATE TARGET GROUP

## Step 9: Create Target Group

### Name:

```text
web-servers-tg
```

### Target Type:

✔ Instance

### Protocol:

✔ HTTP

### Port:

✔ 80

### Health Check Path:

```text
/
```

---

## Step 10: Register Targets

### Select:

✔ webserver-1
✔ webserver-2

### Click:

**Include as pending below** → Register

---

## Step 11: Create Load Balancer

### Review → Click:

**Create Load Balancer**

---

## Output:

✔ ALB provisioned successfully

---

# PART E: VERIFY LOAD BALANCER

## Step 12:

1. Open Load Balancer details
2. Copy DNS name
3. Open browser

---

## Expected:

✔ Displays Server 1 or Server 2 page
✔ Refresh shows traffic switching between servers

---

# Step 13: Verify Health Checks

1. Target Groups → Targets

### Expected:

✔ Both instances healthy

---

# PART F: CREATE AMI FOR AUTO SCALING

## Step 14: Create Image

1. EC2 → Instances
2. Select configured webserver instance
3. Actions → Image and templates → Create image

### Name:

```text
my-webserver-AMI
```

### Click:

**Create image**

---

## Wait:

✔ AMI status = Available

---

# PART G: CREATE LAUNCH TEMPLATE

## Step 15:

1. EC2 → Launch Templates
2. Click:
   **Create Launch Template**

### Name:

```text
my-launch-template
```

### AMI:

✔ my-webserver-AMI

### Instance Type:

✔ t2.micro

### Key Pair:

✔ Existing key pair

### Security Group:

✔ Web server security group (HTTP + SSH)

### Click:

**Create Launch Template**

---

## Output:

✔ Launch Template ready

---

# PART H: CREATE AUTO SCALING GROUP

## Step 16:

1. EC2 → Auto Scaling Groups
2. Click:
   **Create Auto Scaling Group**

### Name:

```text
webserver-asg
```

### Launch Template:

✔ my-launch-template

---

## Step 17: Network Settings

### VPC:

✔ Default VPC

### Subnets:

✔ Select all available subnets

---

## Step 18: Attach Load Balancer

### Choose:

✔ Attach to existing load balancer

### Select:

```text
my-alb
```

### Health Check Type:

✔ ELB + HTTP

### Grace Period:

```text
300 seconds
```

---

## Step 19: Group Size

### Desired Capacity:

```text
2
```

### Minimum Capacity:

```text
1
```

### Maximum Capacity:

```text
4
```

---

## Step 20: Review & Create

### Click:

**Create Auto Scaling Group**

---

## Output:

✔ ASG successfully configured

---

# PART I: VERIFY AUTO SCALING

## Step 21:

1. EC2 → Auto Scaling Groups → Instances

### Expected:

✔ Minimum 2 healthy instances running

---

## Step 22: Test Auto Recovery

1. Manually terminate one instance

---

## Expected:

✔ ASG automatically launches replacement instance

---

## Step 23: Verify Load Balancer

### Refresh ALB DNS repeatedly:

✔ Traffic balanced across active instances

---

# PART J: OPTIONAL SCALING POLICY

## Step 24:

Configure dynamic scaling based on:
✔ CPU > 70%

### Service:

Amazon CloudWatch alarms

---

# PART K: REQUIRED SUBMISSION PROOF

## Recommended Screenshots:

✔ EC2 webserver-1
✔ EC2 webserver-2
✔ Apache outputs
✔ Security groups
✔ ALB creation
✔ Target group healthy status
✔ ALB DNS output
✔ AMI creation
✔ Launch Template
✔ Auto Scaling Group
✔ Replacement instance after termination

---

# PART L: CLEANUP

## Delete:

✔ Auto Scaling Group
✔ Launch Template
✔ Load Balancer
✔ Target Group
✔ EC2 instances
✔ AMI
✔ Security groups
✔ Key pairs if unnecessary

---

# FINAL OUTPUT:

✔ Two EC2 web servers deployed
✔ Apache configured successfully
✔ ALB distributing traffic
✔ Target group healthy
✔ AMI created
✔ Launch Template configured
✔ Auto Scaling Group working
✔ Fault tolerance verified
✔ High availability achieved

---

# Viva Questions:

## 1. What is Elastic Load Balancer?

Distributes incoming traffic across multiple EC2 instances.

## 2. What is Auto Scaling?

Automatically adjusts EC2 instance count based on demand.

## 3. Why use Target Groups?

To manage registered backend servers.

## 4. What are Health Checks?

Detect unhealthy instances and stop traffic routing.

## 5. Why create an AMI?

To replicate configured instances automatically.

## 6. What is Launch Template?

Defines instance launch configuration for ASG.

## 7. What is desired capacity?

Preferred number of running instances.

## 8. What happens if an instance fails?

ASG replaces it automatically.

---

# Common Errors:

## 1. Target unhealthy:

* Apache not running
* Security group HTTP blocked

## 2. ALB DNS not loading:

* Wrong SG
* Wrong target group

## 3. ASG not launching:

* Launch template issue
* Capacity limits

## 4. Scaling not triggering:

* Missing CloudWatch alarms

---

# Final Conclusion:

The Week 9 practical successfully demonstrates AWS Elastic Load Balancing, Auto Scaling, high availability architecture, traffic distribution, fault tolerance, and scalable cloud infrastructure design using EC2, ALB, ASG, Launch Templates, and CloudWatch. fileciteturn14file0


========================================================================================================
================================================================================================================

#week 10 elastic bean stalk

 create application->

MyApplication

platform->node,branch->20
application code->upload your code->local file->version1

next

##configure service access

PART A: CREATE SERVICE ROLE
Click:
Create role

(next to Service role)

In IAM:
Trusted entity:

✔ AWS Service

Use case:

✔ Elastic Beanstalk

Attach policies:
AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy
AWSElasticBeanstalkEnhancedHealth
Role name:
aws-elasticbeanstalk-service-role
Click:
Create role
PART B: CREATE INSTANCE PROFILE ROLE
Again click:
Create role

(next to EC2 instance profile)

Trusted entity:

✔ AWS Service

Use case:

✔ EC2

Attach policies:
AWSElasticBeanstalkWebTier
AWSElasticBeanstalkWorkerTier
AWSElasticBeanstalkMulticontainerDocker
Role name:
aws-elasticbeanstalk-ec2-role
Click:
Create role
PART C: RETURN TO ELASTIC BEANSTALK
Click refresh icon beside dropdowns
Now choose:
Service role:
aws-elasticbeanstalk-service-role
Instance profile:
aws-elasticbeanstalk-ec2-role

click next

##setup networking

default vpc 
public ip enable 
select all availability zones

next

## configure instance traffic

IMDS ->default
next

##configure updates

next

##review 
create

##go to applications -MyApplications click domain

==============================================================================
=================================================================================================

# WEEK 11: AMAZON LONEX (AMAZON LEX) HOTEL BOOKING CHATBOT PRACTICAL RECORD

## Aim:

To create and test a Hotel Booking Chatbot using Amazon Lex by configuring bot creation, intents, utterances, slots, conditional branching, custom slot types, response cards, confirmation prompts, testing, and resource cleanup.

---

# PART A: AWS LOGIN AND AMAZON LEX SETUP

## Step 1: Login to AWS Console

1. Open your browser.
2. Go to AWS Console login page.
3. Sign in using your AWS account credentials.
4. After successful login, ensure your AWS region is set to:

   * **US East (N. Virginia) – us-east-1**
   * This is located in the top-right corner of the AWS Console.

---

## Step 2: Open Amazon Lex

1. In the top search bar, type:

   ```
   Amazon Lex
   ```
2. Select **Amazon Lex V2** from search results.

---

# PART B: BOT CREATION

## Step 3: Create Bot

1. On Amazon Lex homepage, click:
   **Create bot** (top-right corner)
2. Select:
   **Create a blank bot**
3. Enter the following:

   * **Bot name:** `HotelBookingBot`
4. IAM Permissions:

   * Select **Create a role with basic Amazon Lex permissions**
5. COPPA:

   * Select **No**
6. Language:

   * English (US)
7. Session timeout:

   * Keep default settings
8. Click:
   **Done**

### Note:

If IAM role suffix conflict appears, AWS auto-generates a unique role. Proceed with the generated role.

---

# PART C: INTENT CREATION

## Step 4: Create Intent

1. After bot creation, Lex opens the default intent page.
2. Rename `NewIntent` to:

   ```
   BookHotel
   ```
3. Click:
   **Save intent**

---

# PART D: SAMPLE UTTERANCES

## Step 5: Add Sample Utterances

Under **Sample utterances**, add the following one by one:

* I want to book a hotel
* Book a room
* Reserve hotel
* I need a room

### After each:

Click **+ Add utterance**

### Then:

Click **Save intent**

---

# PART E: SLOT CREATION

# SLOT 1: AGE

## Step 6: Create Age Slot

1. Click **Add slot**
2. Fill:

   * **Slot name:** `age`
   * **Slot type:** `AMAZON.Number`
   * **Prompt:** `What is your age?`
3. Enable:

   * **Required for this intent**
4. Click:
   **Save slot**

---

## Step 7: Add Conditional Branching for Age Restriction

1. Open age slot.
2. Scroll to:
   **Slot capture: success response**
3. Click:
   **Add conditional branching**
4. Enter condition:

   ```
   {age} < 18
   ```
5. Response:

   ```
   You are not eligible for hotel booking.
   ```
6. Next step:
   **End conversation**
7. Click:
   **Update slot**

---

# SLOT 2: LOCATION

## Step 8: Create Location Slot

1. Click **Add slot**
2. Fill:

   * **Slot name:** `location`
   * **Slot type:** `AMAZON.City`
   * **Prompt:** `Which city do you want?`
3. Enable Required
4. Save slot

---

# SLOT 3: CHECK-IN DATE

## Step 9: Create Check-in Slot

1. Click **Add slot**
2. Fill:

   * **Slot name:** `checkin`
   * **Slot type:** `AMAZON.Date`
   * **Prompt:** `What is your check-in date?`
3. Retry prompt:

   ```
   Please provide a valid date.
   ```
4. Save slot

---

# SLOT 4: NUMBER OF NIGHTS

## Step 10: Create Nights Slot

1. Click **Add slot**
2. Fill:

   * **Slot name:** `nights`
   * **Slot type:** `AMAZON.Number`
   * **Prompt:** `How many nights will you stay?`
3. Enable Required
4. Save slot

---

# PART F: CUSTOM SLOT TYPE CREATION

## Step 11: Save Intent

Click:
**Save intent**

---

## Step 12: Create RoomType Slot Type

1. Go to left sidebar → **Slot types**
2. Click:
   **Create slot type**
3. Choose blank slot type
4. Enter:

   * **Slot type name:** `RoomType`
5. Add values:

   * Single
   * Double
   * Suite
6. Save slot type

---

# PART G: ROOM TYPE SLOT

## Step 13: Add Room Type Slot

1. Return to:
   **BookHotel intent**
2. Click **Add slot**
3. Fill:

   * **Slot name:** `roomtype`
   * **Slot type:** `RoomType`
   * **Prompt:** `Which room type do you prefer?`
4. Enable Required

---

## Step 14: Add Response Card Buttons

1. Under Slot prompts, click:
   **More prompt options**
2. Add card group

### Fill:

* **Title:** `Select Room Type`
* **Subtitle:** `Choose your preferred room option`

### Add Buttons:

#### Button 1:

* Title: `Single`
* Value: `Single`

#### Button 2:

* Title: `Double`
* Value: `Double`

#### Button 3:

* Title: `Suite`
* Value: `Suite`

3. Click:
   **Update prompts**
4. Click:
   **Update slot**

---

# PART H: CONFIRMATION SETTINGS

## Step 15: Configure Confirmation Prompt

Scroll to **Confirmation** section.

### Confirmation prompt:

```
Do you want to confirm booking in {location} for {nights} nights?
```

### Decline response:

```
Booking cancelled.
```

### Success response:

```
Booking confirmed.
```

---

## Step 16: Save Intent

Click:
**Save intent**

---

# PART I: BUILD BOT

## Step 17: Build Bot

1. Top-right corner:
   Click **Build**
2. Wait until:
   **Successfully built** status appears.

---

# PART J: TEST BOT

## Step 18: Open Test Panel

1. Top-right:
   Click **Test**

---

## Step 19: Sample Testing Flow

### User:

```
Book a hotel
```

### Bot:

```
What is your age?
```

### User:

```
25
```

### Bot:

```
Which city do you want?
```

### User:

```
Hyderabad
```

### Bot:

```
What is your check-in date?
```

### User:

```
Tomorrow
```

### Bot:

```
How many nights will you stay?
```

### User:

```
2
```

### Bot:

```
Which room type do you prefer?
```

### User:

```
Single / Double / Suite
```

### Bot:

```
Do you want to confirm booking in Hyderabad for 2 nights?
```

### User:

```
Yes
```

### Bot:

```
Booking confirmed.
```

---

# PART K: OUTPUT

## Expected Result:

The chatbot successfully:

* Validates age
* Restricts underage users
* Captures city
* Captures check-in date
* Captures stay duration
* Captures room type
* Requests booking confirmation
* Confirms booking

---

# Viva Questions:

## 1. What is Amazon Lex?

Amazon Lex is an AWS service used to build conversational chatbots using voice and text.

## 2. What is an Intent?

An intent defines the user’s goal or action.

## 3. What is a Slot?

A slot collects user-specific information required for fulfilling an intent.

## 4. What is Conditional Branching?

Conditional branching controls chatbot flow based on slot values.

## 5. What are Response Cards?

Response cards are button-based quick reply options.

---

# Final Conclusion:

The Amazon Lex HotelBookingBot was successfully created, configured, tested, and validated with full conversational flow and booking confirmation.






============================================================================================
====================================================================================================


# WEEK 12: IAM USER MANAGEMENT AND AWS CLI PRACTICAL RECORD

## Aim:

To create IAM users with restricted permissions, test console access restrictions, generate access keys, configure AWS CLI, verify policy enforcement, and perform S3 operations securely.

---

# PART A: ROOT USER LOGIN AND IAM SETUP

## Step 1: Login as Root User

1. Open browser.
2. Visit AWS Console login page.
3. Select:
   **Root user**
4. Enter:

   * Root email address
   * Root password
5. Complete MFA if required.

---

## Step 2: Open IAM Dashboard

1. In AWS top search bar, type:

   ```
   IAM
   ```
2. Select:
   **Identity and Access Management (IAM)**

---

# PART B: CREATE IAM USER

## Step 3: Open Users Section

1. Left sidebar:
   Click **Users**
2. Top-right:
   Click **Create user**

---

## Step 4: Enter User Details

### User name:

```
S3_Specialist
```

### Enable:

✔ Provide user access to AWS Management Console

### Password:

Select:
**Custom password**

### Example password:

```
S3User@123
```

### Password reset:

Optional based on lab requirement

### Click:

**Next**

---

# PART C: ASSIGN PERMISSIONS

## Step 5: Attach Direct Policy

1. Select:
   **Attach policies directly**
2. Search:

   ```
   AmazonS3FullAccess
   ```
3. Check:
   ✔ AmazonS3FullAccess
4. Click:
   **Next**

---

## Step 6: Review and Create User

### Verify:

* Username correct
* Console access enabled
* Password set
* S3 policy attached

### Click:

**Create user**

---

## Step 7: Download User Credentials CSV

1. On success page:
   Click **Download .csv file**
2. Save securely.

### CSV contains:

* Sign-in URL
* Username
* Password
* Account ID

---

# PART D: LOGIN AS IAM USER

## Step 8: Sign Out Root User

1. Top-right account menu
2. Click:
   **Sign out**

---

## Step 9: Login Using IAM Credentials

### Use:

* Account ID
* IAM username:

```
S3_Specialist
```

* Password from CSV
* Sign-in URL

---

# PART E: TEST RESTRICTED PERMISSIONS

## Step 10: Test EC2 Access

1. Search:

   ```
   EC2
   ```
2. Open EC2 dashboard

### Expected:

❌ API errors / AccessDenied

### Purpose:

Confirms EC2 access is restricted.

---

## Step 11: Test S3 Access

1. Search:

   ```
   S3
   ```
2. Open Amazon S3
3. Click:
   **Create bucket**

### Bucket name example:

```
naveen-week12-bucket-4499
```

### Important:

Bucket name must be globally unique.

4. Leave defaults
5. Click:
   **Create bucket**

### Expected:

✔ Bucket created successfully

---

# PART F: GENERATE ACCESS KEYS FOR CLI

## Step 12: Login Back as Root/Admin

1. Return to root user
2. Open IAM → Users → S3_Specialist

---

## Step 13: Open Security Credentials

1. Click:
   **Security credentials** tab
2. Scroll to:
   **Access keys**
3. Click:
   **Create access key**

---

## Step 14: Select CLI Use Case

1. Choose:
   **Command Line Interface (CLI)**
2. Confirm checkbox
3. Click:
   **Next**

---

## Step 15: Set Description Tag

### Enter:

```
AWS CLI Access for S3 Specialist
```

### Click:

**Create access key**

---

## Step 16: Download Access Key CSV

### Save securely.

### Contains:

* Access Key ID
* Secret Access Key

### Important:

Secret key is shown only once.

---

# PART G: AWS CLI INSTALLATION

## Step 18: Install AWS CLI

https://awscli.amazonaws.com/AWSCLIV2.msi
---

## Step 19: Verify Installation

1. Open Command Prompt
2. Run:

```bash
aws --version
```

### Expected:

```bash
aws-cli/2.x.x
```

---

# PART H: CONFIGURE AWS CLI

## Step 20: Configure CLI

Run:

```bash
aws configure
```

### Enter:

### Access Key ID:

(from CSV)

### Secret Access Key:

(from CSV)

### Default region:

```bash
ap-south-1
```

### Output format:

```bash
json
```

---

# PART I: TEST CLI RESTRICTIONS

## Step 21: Test EC2 via CLI

```bash
aws ec2 describe-instances
```

### Expected:

❌ AccessDenied

### Purpose:

Confirms policy restrictions.

---

## Step 22: Test S3 via CLI

```bash
aws s3 ls
```

### Expected:

✔ Lists S3 buckets

---

# PART J: CREATE S3 BUCKET VIA CLI

## Step 23: Create Bucket

### Example:

```bash
aws s3 mb s3://naveen-week12-bucket-4499
```

### Expected:

```bash
make_bucket: naveen-week12-bucket-4499
```

---

## Step 24: Verify in AWS Console

1. Open S3 dashboard
2. Confirm new bucket exists

---

# PART K: OUTPUT

## Expected Results:

✔ IAM user created successfully
✔ EC2 restricted
✔ S3 permitted
✔ CLI configured
✔ Policy restrictions validated
✔ S3 bucket created successfully via console and CLI
---

# Viva Questions:

## 1. What is IAM?

AWS service for secure user and permission management.

## 2. Why avoid root user?

Root has unrestricted access and high security risk.

## 3. What is AmazonS3FullAccess?

Policy granting complete access to S3 resources.

## 4. What is AWS CLI?

Command-line interface for managing AWS services.

## 5. What does `aws configure` do?

Stores credentials and region settings for CLI use.

## 6. What is least privilege?

Providing only required permissions.

---

# Final Conclusion:

The Week 12 IAM and AWS CLI practical successfully demonstrates secure identity management, permission enforcement, programmatic AWS access, and S3 resource administration using both AWS Console and CLI.


===================================================================================
===========================================================================================

# WEEK 13: IAM ROLES FOR EC2 INSTANCE PRACTICAL RECORD

## Aim:

To create an IAM Role for EC2, assign AmazonS3FullAccess permissions, attach the role to an EC2 instance, verify temporary credential-based access to S3, validate restricted permissions for other AWS services, and understand secure role-based access management.

---

# PART A: LOGIN TO AWS CONSOLE

## Step 1: Login to AWS Console

1. Open browser.
2. Go to AWS Console login page.
3. Login using Root/Admin credentials.
4. Select preferred region:

   * **ap-south-1** (Mumbai) or lab-specific region.

---

# PART B: CREATE IAM ROLE

## Step 2: Open IAM Dashboard

1. Top search bar:

   ```
   IAM
   ```
2. Select:
   **Identity and Access Management (IAM)**

---

## Step 3: Open Roles Section

1. Left sidebar:
   Click **Roles**
2. Top-right:
   Click **Create role**

---

## Step 4: Select Trusted Entity

1. Choose:
   **AWS service**
2. Under service use case:
   Select:
   **EC2**

### Purpose:

Allows EC2 instances to assume this role securely.

3. Click:
   **Next**

---

## Step 5: Assign Permissions

1. Search:

   ```
   AmazonS3FullAccess
   ```
2. Select:
   ✔ AmazonS3FullAccess
3. Click:
   **Next**

---

## Step 6: Name and Create Role

### Role name:

```
EC2-S3-ReadOnly-Role
```

### Description:

```
Allows EC2 instance to access Amazon S3 securely without access keys.
```

### Click:

**Create role**

---

## Output:

✔ IAM Role successfully created.

---

# PART C: CREATE EC2 INSTANCE

## Step 7: Open EC2 Dashboard

1. Search:

   ```
   EC2
   ```
2. Click:
   **EC2 Dashboard**

---

## Step 8: Launch New Instance

1. Click:
   **Launch instance**

### Enter:

### Name:

```
Week13-EC2-Role-Test
```

### AMI:

✔ Amazon Linux 2

### Instance Type:

✔ t2.micro (Free Tier)

### Key Pair:

* Create new OR use existing key pair

### Security Group:

✔ Allow SSH (Port 22)

### Click:

**Launch instance**

---

# PART D: CONNECT TO EC2 INSTANCE

## Method 1: EC2 Instance Connect (Recommended)

### Step 9:

1. EC2 Dashboard → Instances
2. Select:
   `Week13-EC2-Role-Test`
3. Click:
   **Connect**
4. Select:
   **EC2 Instance Connect**
5. Username:

```
ec2-user
```

6. Click:
   **Connect**

---

## Output:

✔ Browser terminal opens successfully.

---

# PART E: VERIFY NO ACCESS BEFORE ROLE ATTACHMENT

## Step 10: Test S3 Access

Run:

```bash
aws s3 ls
```

### Expected:

❌ AccessDenied / Unable to locate credentials

### Purpose:

Confirms instance has no default S3 permissions.

---

# PART F: ATTACH IAM ROLE TO EC2 INSTANCE

## Step 11: Return to EC2 Dashboard

1. Select running instance
2. Top menu:
   Click **Actions**
3. Navigate:
   **Security → Modify IAM role**

---

## Step 12: Assign Role

### Select:

```
EC2-S3-ReadOnly-Role
```

### Click:

**Update IAM role**

---

## Output:

✔ Role successfully attached.

---

# PART G: VERIFY ROLE FUNCTIONALITY

## Step 13: Reconnect to EC2 Terminal

1. Use EC2 Instance Connect again if needed.

---

## Step 14: Test S3 Access Again

Run:

```bash
aws s3 ls
```

### Expected:

✔ S3 bucket list displayed successfully

### Explanation:

AWS automatically provides temporary security credentials through IAM Role.

---

## Step 15: Verify Restricted EC2 Access

Run:

```bash
aws ec2 describe-instances
```

### Expected:

❌ Unauthorized / AccessDenied

### Reason:

Role has only AmazonS3FullAccess, not EC2 permissions.

---

# PART H: OUTPUT

## Expected Results:

✔ IAM Role created
✔ EC2 instance launched
✔ EC2 connected successfully
✔ S3 access denied before role attachment
✔ Role attached successfully
✔ S3 access successful after role attachment
✔ EC2 permissions still restricted
✔ Temporary credentials validated

---

# PART I: SECURITY BENEFITS

✔ No Access Key or Secret Key required
✔ Temporary credentials automatically managed
✔ Secure service-to-service access
✔ Reduced credential leakage risk
✔ Follows least privilege principle


# Viva Questions:

## 1. What is an IAM Role?

A temporary permission set assumed by AWS services.

## 2. Why use IAM Role instead of Access Keys?

Roles provide temporary credentials securely without storing long-term secrets.

## 3. What is a trusted entity?

The AWS service allowed to assume the role (e.g., EC2).

## 4. Why did `aws s3 ls` work after role attachment?

Because EC2 received temporary S3 credentials from IAM role.

## 5. Why did `aws ec2 describe-instances` fail?

Because the role only had S3 permissions.

## 6. Difference between IAM User and IAM Role?

* IAM User: Permanent identity with credentials
* IAM Role: Temporary assumable permissions

---

# Common Errors:

## 1. Role not visible during attachment:

* Ensure trusted entity = EC2

## 2. S3 still denied:

* Wait 1–2 minutes for role propagation
* Reconnect EC2

## 3. SSH connection issues:

* Verify security group SSH rule
* Check public IP
* Use correct key pair

---

# Final Conclusion:

The Week 13 practical successfully demonstrates secure IAM Role creation, EC2 trust relationships, temporary credential management, role-based authorization, and secure AWS service access without hardcoded credentials.

