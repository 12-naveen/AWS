first week
1. Creation of Amazon EC2 Instance (Linux)
2. Establishing Secure Connection to Linux EC2 using SSH (Local Machine)
3. Connecting to Linux EC2 using PuTTY (.pem to .ppk Conversion)
4. Connecting to Linux EC2 using AWS CloudShell
5. Creation of Amazon EC2 Instance (Windows)
6. Retrieving Windows Administrator Password using Key Pair
7. Connecting to Windows EC2 using Remote Desktop Protocol (RDP)

1. Creation of Amazon EC2 Instance (Linux)
Scroll down to Application and OS Images (AMI)

👉 Select Ubuntu Server 22.04 LTS
(It will show Free tier eligible if available)

Name: ubuntu-key
Key type: RSA
Private key file format: .pem
Click Create key pair

Step 6: Network Settings
Scroll to Network Settings
Click Edit
Make sure:
Auto assign public IP → Enabled
Security Group → Allow SSH traffic
Type: SSH
Port: 22
Source: My IP
This is VERY important. Without this SSH won’t work.  launch instance

2. Establishing Secure Connection to Linux EC2 using SSH (Local Machine)
Now select your instance.
On top right:👉 Click Connect
You’ll see multiple tabs:EC2 Instance Connect  SSH Client Session Manager

Method 1: Connect Using SSH (Command Prompt)
Click SSH Client tab
You’ll see command like:
ssh -i "ubuntu-key.pem" ubuntu@ec2-xx-xx-xx.compute.amazonaws.com
Copy this.
Now:Open Command Prompt
Go to Downloads folder (where .pem file is)
Paste the command
Press Enter
Type yes when asked.
You are connected.
enter whoami


🔹 PART 3: Connect Using AWS CloudShell
Top right corner of AWS Console:
👉 Click Terminal icon (CloudShell)
CloudShell opens.
Now go to:EC2 → Select instance → Connect → SSH client
Copy the ssh command.
In CloudShell:
First upload key:
👉 Click Actions (+ icon)
👉 Upload file
👉 Upload .pem
After upload, run this command inside CloudShell: chmod 400 keypair1.pem
Then run the ssh command.
Connected.   if not working chamge my ip to anywhere from security and edit inbound rules
run whoami

5. Creation of Amazon EC2 Instance (Windows)
Step 3: Choose AMI (Operating System)
Under Application and OS Images (AMI):
Select:👉 Microsoft Windows Server 2022 Base
Make sure it shows architecture: 64-bit.

| Type | Protocol | Port | Source |
| ---- | -------- | ---- | ------ |
| RDP  | TCP      | 3389 | My IP  |

PART 2: Get Windows Administrator Password
Now select the Windows instance (checkbox).
Click:👉 Connect (top right)
Go to:👉 RDP Client tab
Click:👉 Get Password
Now:Upload your keypair1.pem
Click Decrypt Password
You will see:
Username:
Administrator
Password:
(random generated password)

Copy that password.

🔹 PART 3: Connect Using Remote Desktop (RDP)

Now go to your local Windows system.

Press:

Windows + R

Type:

mstsc

Press Enter.

Remote Desktop window opens.

In the Computer field:

Paste:

👉 Public IPv4 address of Windows EC2

Click Connect

You’ll see security warning → Click Yes

Login Screen Appears

Username:

Administrator

Password:
Paste the decrypted password

Click OK.
---

week-2

1(a). Create and attach an EBS volume to an EC2 instance and scale the instance by changing the instance type (increase/decrease CPU and RAM)
1(b). Increasing/Decreasing CPU and RAM by modifying the instance type

Step 1: Stop the Instance

Go to AWS Console

Click EC2

Click Instances

Select your Linux instance

Click:

👉 Instance state
👉 Stop instance

Wait until:

State → Stopped

Step 2: Change Instance Type

While instance is stopped:

Click Actions

Click Instance settings

Click Change instance type

You’ll see current type:

t2.micro

Now select a higher one like:

t2.small

or

t2.medium

Click:

👉 Apply

Step 3: Start the Instance

Now click:

👉 Instance state
👉 Start instance

Wait until it becomes:

Running
3/3 checks passed

Step 4: Verify Change

Select the instance.

Check the column:

👉 Instance type

It should now show:

t2.small

(or whatever you selected)

That means CPU and RAM increased.

🔥 Important Viv

2. Attach and permanently mount an EBS volume to a Linux EC2 instance to ensure data persistence across reboots

Week-2 Task 2
Attach and Permanently Mount EBS Volume (Linux)
Step 1: Verify Attached Disk
lsblk

(New disk appears as nvme1n1)

Step 2: Create Partition
sudo fdisk /dev/nvme1n1

Inside fdisk:

n → p → Enter → Enter → Enter → w

Reload partition table:

sudo partprobe

Verify:

lsblk

(New partition appears as nvme1n1p1)

Step 3: Format the Partition
sudo mkfs.xfs /dev/nvme1n1p1

Verify:

lsblk -fs
Step 4: Create Mount Directory
sudo mkdir /mnt/naveen
Step 5: Mount the Volume
sudo mount /dev/nvme1n1p1 /mnt/naveen

Verify:

df -h
Step 6: Make Mount Permanent (Persistence)

Get UUID:

sudo blkid /dev/nvme1n1p1

Edit fstab:

sudo nano /etc/fstab

Add line:

UUID=<your-uuid>   /mnt/naveen   xfs   defaults,nofail   0   0

Test:

sudo mount -a

Reboot:

sudo reboot

Verify after reboot:

df -h
Final Result

EBS volume mounts automatically after reboot → Data persistence ensured.
3. Create a snapshot of an attached EBS volume, copy it to another AWS region, create a new volume from the snapshot, and attach it to an EC2 instance in the destination region
🔹 Step 1: Create Snapshot

Go to AWS Console

Click EC2

Click Volumes (left side)

Select your 10GB volume (nvme1n1 one).

Click:

👉 Actions
👉 Create snapshot

Fill:

Name: week2-snapshot

Description: EBS snapshot

Click:

👉 Create snapshot

Now go to:

👉 Snapshots (left side)

Wait until Status → Completed

🔹 Step 2: Copy Snapshot to Another Region

Select your snapshot.

Click:

👉 Actions
👉 Copy snapshot

Now:

Destination region → Choose a different one
(Example: if current is us-east-1, choose us-west-2)

Click:

👉 Copy snapshot

This may take some time.

🔹 Step 3: Switch Region

Top right of AWS Console:

Click region dropdown.

Switch to the destination region you selected.

Now go to:

👉 EC2 → Snapshots

You should see the copied snapshot there.

Wait until Status → Completed

🔹 Step 4: Create Volume from Snapshot

Select the copied snapshot

Click Actions

Click Create volume

Important:

Choose Availability Zone of an EC2 instance in this new region.

Click:

👉 Create volume

🔹 Step 5: Attach Volume to EC2

Go to:

👉 Volumes

Select newly created volume.

Click:

👉 Actions → Attach volume

Choose:

Instance in that region

Device name (default is fine)

Click Attach.

🔹 Step 6: Verify in Linux

SSH into that EC2.

Run:

lsblk

You should see new NVMe disk.

Mount it if needed:

sudo mount /dev/nvme1n1p1 /mnt/test

All old data will be there.

=======================================================================================================================

week 3

1. Launch two Amazon Linux EC2 instances (EFS-1 and EFS-2) in different subnets
2. Create an Amazon Elastic File System (EFS)
3. Access both EC2 instances and install Amazon EFS utilities
4. Attach (mount) the EFS file system to both EC2 instances
5. Verify and test file synchronization between the two instances

1. Launch two Amazon Linux EC2 instances (EFS-1 and EFS-2) in different subnets
Launch two Amazon Linux EC2 instances (EFS-1 and EFS-2) in different subnets

When this is fully done, I’ll clearly tell you it’s completed.

🔹 Step 1: Go to EC2

Open AWS Console

Click Services

Click EC2

Click Launch Instance

🔹 Step 2: Configure First Instance (EFS-1)
1️⃣ Name
EFS-1
2️⃣ AMI (Important)

Choose:
👉 Amazon Linux 2
(Not Ubuntu — EFS utilities work easily with Amazon Linux)

3️⃣ Instance Type

👉 t2.micro

4️⃣ Key Pair

Select your existing key pair

5️⃣ Network Settings

Click Edit

Now choose:

VPC → Default

Subnet → Subnet-1a

Auto-assign public IP → Enabled

6️⃣ Security Group

Create new security group:

Add these rules:

Type	Port	Source
SSH	22	My IP
NFS	2049	Anywhere

(NFS is required for EFS)

Click:

👉 Launch Instance

🔹 Step 3: Launch Second Instance (EFS-2)

Click Launch Instance again.

Now configure:

1️⃣ Name
EFS-2
2️⃣ AMI

👉 Amazon Linux 2

3️⃣ Instance Type

👉 t2.micro

4️⃣ Key Pair

Same key pair

5️⃣ Network Settings

Click Edit

VPC → Default

Subnet → Subnet-1b (different subnet from first instance)

Auto-assign public IP → Enabled

6️⃣ Security Group

Use the same security group (with SSH + NFS)

Click:

👉 Launch Instance

🔹 Step 4: Wait for Both Instances

Go to:

EC2 → Instances

Wait until:

✔ Both EFS-1 and EFS-2
✔ Status = Running
✔ 2/2 checks passed

✅ Task Completion Condition

This task is complete only when:

Two instances exist

Both are Amazon Linux

They are in different subnets

Both are running

------------------------------------------------------------------------------------------------

2. Create an Amazon Elastic File System (EFS)

Create an Amazon Elastic File System (EFS)

We’ll do this properly and clean.

🔹 Step 1: Go to EFS Service

Open AWS Console

Click Services

Search and click EFS (Elastic File System)

You’ll enter the EFS dashboard.

🔹 Step 2: Create File System

Click:

👉 Create file system

🔹 Step 3: Configure Basic Settings

Fill:

Name → EFS-Lab

VPC → Default (same VPC as your EC2 instances)

Leave other settings default.

Click:

👉 Next

🔹 Step 4: Network Settings (Very Important)

You’ll see mount targets for Availability Zones.

For each AZ:

Remove default security group

Select the security group you created earlier (the one with NFS port 2049 allowed)

This is important because:

EFS works on port:

2049 (NFS)

Make sure:

✔ Both Subnet-1a and Subnet-1b have mount targets
✔ Security group allows NFS

Click:

👉 Next

🔹 Step 5: Review and Create

Review settings.

Click:

👉 Create file system

🔹 Step 6: Wait for Status

Go to:

EFS → File systems

Wait until:

Status = Available

✅ Task Completion Condition

This task is completed when:

EFS file system is created

Mount targets exist in both subnets

Status shows Available

-----------------------------------------------------------------------------------------------
3. Access both EC2 instances and install Amazon EFS utilities

🔹 Step 1: Open Two Terminals

Open two PowerShell windows (or two CMD windows).

We will connect:

Terminal 1 → EFS-1

Terminal 2 → EFS-2

🔹 Step 2: SSH into EFS-1

In first terminal:

ssh -i keypair.pem ec2-user@<EFS-1-public-ip>

If Amazon Linux asks yes/no → type yes.

You should see:

[ec2-user@ip-xxx-xxx ~]$
🔹 Step 3: Switch to Root

Run:

sudo su

Prompt changes to:

[root@ip-xxx ~]#
🔹 Step 4: Create Directory

Run:

mkdir /efs
🔹 Step 5: Install EFS Utilities

Run:

yum install -y amazon-efs-utils

Wait until installation completes.

Verify (optional):

rpm -qa | grep efs
🔹 Now Repeat for EFS-2

In second terminal:

SSH into EFS-2

Run sudo su

Create directory mkdir /efs

Install:

yum install -y amazon-efs-utils
✅ Task Completion Condition

This task is complete when:

Both EFS-1 and EFS-2 are accessible via SSH

Both are root user

Both have /efs directory created

Both have amazon-efs-utils installed

--------------------------------------------------------------------------------------------------
4. Attach (mount) the EFS file system to both EC2 instances

🔹 Step 1: Get Mount Command from EFS

Go to AWS Console

Open EFS

Click your file system (EFS-Lab)

Click Attach (top right)

Choose:

👉 Mount via DNS

You will see a command like:

sudo mount -t efs fs-12345678:/ /efs

Copy that full command.

🔹 Step 2: Mount on EFS-1

Go to your EFS-1 terminal (where you are root).

Paste the mount command.

Example:

sudo mount -t efs fs-xxxx:/ /efs

Press Enter.

If no error appears → mount successful.

Verify:

df -h

You should see something like:

fs-xxxx:/   8.0E   0   8.0E   0%   /efs

That means EFS is mounted.

🔹 Step 3: Mount on EFS-2

Go to EFS-2 terminal.

Switch to root if not:

sudo su

Paste the same mount command.

Verify:

df -h

You should see EFS mounted at /efs.

🔹 Step 4: Test File Sharing

Now in EFS-1:

cd /efs
touch testfile.txt
ls

You should see testfile.txt.

Now go to EFS-2:

cd /efs
ls

You should see the same testfile.txt.

That proves:

✔ Shared storage
✔ Real-time synchronization
✔ Both instances using same file system

----------------------------------------------------------------------------------------------------
5. Verify and test file synchronization between the two instances

5. Verify and Test EFS (File Synchronization)

This is the final step of the EFS experiment.

🔹 Step 1: Go to EFS-1

In the EFS-1 terminal (root):

cd /efs
touch file1.txt
echo "Hello from EFS-1" > file1.txt
ls

You should see:

file1.txt
🔹 Step 2: Check in EFS-2

Now go to EFS-2 terminal:

cd /efs
ls

You should see:

file1.txt

Now verify content:

cat file1.txt

You should see:

Hello from EFS-1

That proves synchronization works.

🔹 Step 3: Test Reverse (Optional but Good)

Now in EFS-2:

touch file2.txt
echo "Hello from EFS-2" > file2.txt

Go back to EFS-1:

ls
cat file2.txt

You should see the file.

✅ Final Result

You have:

✔ Created two EC2 instances
✔ Created EFS
✔ Mounted EFS on both
✔ Verified shared file system
✔ Confirmed real-time file sync


=======================================================================================================================

week 4 

1. Create an S3 Bucket, Upload an Object, and Enable Public Access
2. Enable S3 Versioning and Observe Object Versions
3. Configure Cross-Region Replication (CRR)
4. Configure Static Website Hosting in S3

1. Create an S3 Bucket, Upload an Object, and Enable Public Access
1. Create an S3 Bucket, Upload an Object, and Enable Public Access

Follow exactly. After it works, I’ll confirm completion.

🔹 Step 1: Go to S3

Open AWS Console

Click Services

Click S3

🔹 Step 2: Create Bucket

Click:
👉 Create bucket

Now fill:

Bucket name:

naveen-s3-lab-12345

⚠ Bucket name must be globally unique.
If error appears, add random numbers.

Region:
Keep default (us-east-1)

🔹 Step 3: Disable Block Public Access

Scroll down to:

Block Public Access settings

Uncheck:

☑ Block all public access

It will show warning.

Tick:
✔ I acknowledge that the current settings might result in this bucket and the objects within becoming public.

🔹 Step 4: Create Bucket

Click:
👉 Create bucket

🔹 Step 5: Upload Object

Click your newly created bucket

Click:
👉 Upload

Click:
👉 Add files

Select any file (example: test.txt)

Click:
👉 Upload

🔹 Step 6: Make Object Public

Click the uploaded file

Click:
👉 Actions

Click:
👉 Make public using ACL

Confirm if prompted.

🔹 Step 7: Copy Object URL

Inside object page, scroll down.

Copy the:

Object URL

It will look like:

https://bucket-name.s3.amazonaws.com/test.txt

Open it in browser.

If file downloads or opens → Public access works.

✅ Task Completion Conditions

This task is completed when:

✔ S3 bucket created
✔ Public access block disabled
✔ File uploaded
✔ Object made public
✔ File accessible via browser URL
---------------------------------------------------------------------------------------------------------------------

2. Enable S3 Versioning and Observe Object Versions

We’ll enable versioning, upload new versions of the same file, and verify multiple versions exist.

Follow carefully.

🔹 Step 1: Enable Versioning

Go to S3

Click your bucket

Click Properties

Scroll to Bucket Versioning

Click Edit

Select:
👉 Enable

Click Save changes

🔹 Step 2: Upload New Version of Same File

Now modify your file locally.

Open your text file and add:

Version 2 - Updated Content

Save it.

Now:

Go back to bucket

Click Upload

Upload the same file name again (text.txt.txt)

Confirm overwrite if prompted

This creates Version 2.

🔹 Step 3: View Object Versions

Inside bucket

Turn on:
👉 Show versions (top right toggle)

Now you should see:

Latest version

Previous version

Each with different Version IDs

🔹 Step 4: Verify Version IDs

Click the file → Click Versions tab

You should see multiple versions with unique IDs.

🔥 What Just Happened

Without versioning:
Uploading same file overwrites old file permanently.

With versioning:
AWS keeps all previous versions.

✅ Task Completion Conditions

This task is complete when:

✔ Versioning enabled
✔ Same file uploaded multiple times
✔ Show versions displays multiple versions
✔ Each version has unique Version ID
----------------------------------------------------------------------------------------------------------------------
3. Configure Cross-Region Replication (CRR)

We will:

Create destination bucket (different region)

Enable versioning on both buckets

Create IAM role for replication

Configure replication rule

Test replication

When replication works, I’ll confirm completion.

🔹 Step 1: Create Destination Bucket (Different Region)

Go to S3

Click Create bucket

Bucket name:

mybucket1-cclab-replica

Region:
⚠ Change region to something different
Example:

US West (Oregon) – us-west-2

Keep:
Block Public Access → ON (default is fine)

Click:
👉 Create bucket

🔹 Step 2: Enable Versioning on Destination Bucket

Open the new replica bucket

Go to Properties

Scroll to Bucket Versioning

Click Edit

Enable versioning

Save

🔹 Step 3: Configure Replication on Source Bucket

Now go back to:

Your original bucket:

mybucket1-cclab

Click Management

Scroll to Replication rules

Click:
👉 Create replication rule

🔹 Step 4: Replication Rule Settings

Rule name:

crr-rule

Status:
Enabled

Apply to:
✔ Entire bucket

🔹 Step 5: Choose Destination

Destination bucket:
Select:

mybucket1-cclab-replica

Region will auto-detect (us-west-2)

🔹 Step 6: IAM Role

Choose:
👉 Create new role

AWS will automatically create the required IAM role.

Click:
👉 Save / Create rule

🔹 Step 7: Test Replication

Now upload a NEW file into the source bucket.

Example:

replication-test.txt

Wait 20–60 seconds.

Now open:
Replica bucket (us-west-2)

You should see the file automatically appear there.

🔥 Important

CRR only replicates:
✔ New objects uploaded after rule creation
❌ Old objects will NOT replicate automatically

✅ Task Completion Conditions

This task is complete when:

✔ Destination bucket created in different region
✔ Versioning enabled on both buckets
✔ Replication rule created
✔ New file uploaded
✔ File appears automatically in replica bucket
------------------------------------------------------------------------------------------------------------------

4. Configure Static Website Hosting in S3

We will:

Create a new bucket

Upload index.html and error.html

Enable static website hosting

Make bucket public

Test website endpoint

When it works in browser, I’ll confirm completion.

🔹 Step 1: Create New Bucket for Website

Go to S3 → Create bucket

Bucket name:

naveen-static-website-12345

⚠ Must be globally unique. Add random numbers if needed.

Region:
Keep same (us-east-1)

Block Public Access:
Uncheck Block all public access

Tick acknowledgment box.

Click:
👉 Create bucket

🔹 Step 2: Enable Static Website Hosting

Open the bucket

Go to Properties

Scroll to Static website hosting

Click Edit

Enable:
✔ Enable

Index document:

index.html

Error document:

error.html

Click:
👉 Save changes

🔹 Step 3: Create HTML Files

On your laptop create two files:

index.html
<html>
<head><title>My Website</title></head>
<body>
<h1>Hello from S3 Static Website</h1>
</body>
</html>
error.html
<html>
<body>
<h1>Error Page</h1>
</body>
</html>

Save both.

🔹 Step 4: Upload Files

Go to bucket → Upload

Upload:

index.html

error.html

Click Upload.

🔹 Step 5: Add Bucket Policy (Public Read)

Go to:

Bucket → Permissions → Bucket policy → Edit

Paste:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadWebsite",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::naveen-static-website-12345/*"
    }
  ]
}

Replace bucket name with your exact name.

Click Save.

🔹 Step 6: Copy Website Endpoint

Go to:

Properties → Static website hosting

Copy:

Website endpoint URL

It will look like:

http://bucket-name.s3-website-us-east-1.amazonaws.com

Open it in browser.

✅ Task Completion Conditions

This task is complete when:

✔ Bucket created
✔ Static website hosting enabled
✔ index.html uploaded
✔ Bucket policy added
✔ Website endpoint opens in browser
✔ You see "Hello from S3 Static Website"

=======================================================================================================================
week 5
1. Create a Custom VPC
2. Create Public and Private Subnets
3. Create and Attach an Internet Gateway (IGW)
4. Create and Configure Route Tables (Public & Private)
5. Enable Auto-Assign Public IP for Public Subnet
6. Configure Security Groups (SSH, HTTP, HTTPS)
7. Launch EC2 Instances in Public and Private Subnets
8. Test the Setup

Public EC2 → Internet access

Private EC2 → No direct public access

Private EC2 → Internet access via NAT

1. Create a Custom VPC
🔹 Step 1: Go to VPC Service

Open AWS Console

Click Services

Search and click VPC

You’ll enter the VPC dashboard.

🔹 Step 2: Create VPC

Click:

👉 Create VPC

Choose:

👉 VPC only (not VPC and more)

🔹 Step 3: Enter Details

Fill the following:

Name tag:

Custom-VPC

IPv4 CIDR block:

10.0.0.0/16

Tenancy:

Default

Leave everything else as default.

🔹 Step 4: Create

Click:

👉 Create VPC

You should see a success message.

-------------------------------------------------------------------------------------------------------
2. Create Public and Private Subnets


🔹 Step 1: Go to Subnets

Open VPC

Click Subnets (left sidebar)

Click Create subnet

🔹 Step 2: Create Public Subnet

Select:

VPC → Custom-VPC

Now enter:

Name:

Public-Subnet

Availability Zone:
Choose:

us-east-1a

IPv4 CIDR block:

10.0.1.0/24

Click:

👉 Create subnet

🔹 Step 3: Create Private Subnet

Click Create subnet again.

Select:

VPC → Custom-VPC

Enter:

Name:

Private-Subnet

Availability Zone:
Choose same AZ:

us-east-1a

IPv4 CIDR block:

10.0.2.0/24

Click:

👉 Create subnet

🔹 Step 4: Verify

Go to:

VPC → Subnets

You should now see:

Public-Subnet → 10.0.1.0/24

Private-Subnet → 10.0.2.0/24

Both inside Custom-VPC.

✅ Task Completion Condition

This task is complete when:

Both subnets are created

Both are inside Custom-VPC

CIDR blocks are correct

---------------------------------------------------------------------------------------------------------------------

3. Create and Attach an Internet Gateway (IGW)
🔹 Step 1: Go to Internet Gateways

Open AWS Console

Click VPC

In the left menu click:
👉 Internet Gateways

🔹 Step 2: Create IGW

Click:
👉 Create internet gateway

Enter:

Name tag:

Custom-IGW

Click:
👉 Create internet gateway

You should see a success message.

🔹 Step 3: Attach IGW to VPC

Now:

Select the newly created IGW

Click Actions

Click Attach to VPC

Select your VPC:

vpc1 (10.0.0.0/16)

Click:
👉 Attach internet gateway

🔹 Step 4: Verify

Still in Internet Gateways page, check:

State should show: Attached

VPC column should show: vpc1

✅ Task Completion Confirmation

This task is complete when:

Internet Gateway is created

It is attached to your custom VPC

Status shows “Attached”

-------------------------------------------------------------------------------------------------------
4. Create and Configure Route Tables (Public & Private)

4. Create and Configure Route Tables (Public & Private)

We will:

Create Public Route Table

Create Private Route Table

Add correct routes

Associate correct subnets

When finished, I’ll clearly confirm completion.

🔹 PART A — Create Public Route Table
Step 1: Create Route Table

Go to VPC → Route Tables

Click Create route table

Fill:

Name:

Public-RT

VPC:
Select:

vpc1 (10.0.0.0/16)

Click:
👉 Create route table

Step 2: Add Internet Route

Select Public-RT

Go to Routes tab

Click Edit routes

Click Add route

Fill:

Destination:

0.0.0.0/0

Target:
Select:

Custom-IGW

Click:
👉 Save changes

Now Public-RT has:

10.0.0.0/16 → local

0.0.0.0/0 → IGW

Step 3: Associate Public Subnet

Go to Subnet associations tab

Click Edit subnet associations

Select:

subnet1 (10.0.1.0/24)

Click Save

Now subnet1 is Public.

🔹 PART B — Create Private Route Table
Step 4: Create Private Route Table

Click Create route table

Name:

Private-RT

VPC:

vpc1

Click Create

Step 5: Associate Private Subnet

Select Private-RT

Go to Subnet associations

Click Edit subnet associations

Select:

subnet2 (10.0.2.0/24)

Click Save

Do NOT add 0.0.0.0/0 route here.

Private route table should only have:

10.0.0.0/16 → local

🔥 Final Verification

Go to:

VPC → Route Tables

Check:

Public-RT:

Has 0.0.0.0/0 → IGW

Associated with subnet1

Private-RT:

Only local route

Associated with subnet2
------------------------------------------------------------------------------------------------------------------------------
5. Enable Auto-Assign Public IP for Public Subnet

5. Enable Auto-Assign Public IP for Public Subnet

This ensures EC2 instances launched in the public subnet automatically get a public IP.

🔹 Step 1: Go to Subnets

Open VPC

Click Subnets

Select your Public Subnet
(the one associated with Public-RT — usually 10.0.1.0/24)

🔹 Step 2: Edit Subnet Settings

With the public subnet selected, click:
👉 Actions

Click:
👉 Edit subnet settings

🔹 Step 3: Enable Auto-Assign Public IP

You will see:

☐ Auto-assign public IPv4 address

Tick that checkbox.

Then click:

👉 Save

🔹 Step 4: Verify

In the subnet list, check the column:

Auto-assign public IPv4

It should now show:

Yes

-------------------------------------------------------------------------------------------------------------
6. Configure Security Groups (SSH, HTTP, HTTPS)

6. Configure Security Groups (SSH, HTTP, HTTPS)

We will create one security group for the Public instance.

When done, I’ll confirm completion.

🔹 Step 1: Go to Security Groups

Open VPC

Click Security Groups

Click Create security group

🔹 Step 2: Create Public Security Group

Fill:

Name:

Public-SG

Description:

Allow SSH, HTTP, HTTPS

VPC:
Select:

vpc1
🔹 Step 3: Add Inbound Rules

Click Add rule and add these 3 rules:

Rule 1 – SSH

Type:

SSH

Port:

22

Source:

My IP
Rule 2 – HTTP

Type:

HTTP

Port:

80

Source:

Anywhere-IPv4 (0.0.0.0/0)
Rule 3 – HTTPS

Type:

HTTPS

Port:

443

Source:

Anywhere-IPv4 (0.0.0.0/0)

Leave outbound rules as default (Allow all).

Click:

👉 Create security group

🔹 Step 4: Verify

Go back to Security Groups list.

Check:

Public-SG exists

Inbound rules show:

22 → My IP

80 → Anywhere

443 → Anywhere

✅ Task Completion Status

If all three rules are added correctly:

----------------------------------------------------------------------------------------------------------------------

7. Launch EC2 Instances in Public and Private Subnets
A. Launch Public EC2 Instance
Step 1: Go to EC2

EC2 → Instances → Launch Instance

Step 2: Configure

Name:

Public-EC2

AMI:

Amazon Linux 2023

Instance type:

t2.micro

Key pair:
Select your existing key pair

Step 3: Network Settings

Click Edit:

VPC:

vpc1

Subnet:

Public-Subnet (10.0.1.0/24)

Auto-assign public IP:

Enabled

Security group:
Use the security group configured with:

SSH (22)

HTTP (80)

HTTPS (443)

Click:
Launch Instance

B. Launch Private EC2 Instance

Click Launch Instance again.

Name:

Private-EC2

AMI:
Amazon Linux 2023

Instance type:
t2.micro

Key pair:
Same key pair

Network Settings

VPC:

vpc1

Subnet:

Private-Subnet (10.0.2.0/24)

Auto-assign public IP:

Disabled

Security group:
Same security group

Click:
Launch Instance

Verification

After launch:

Public-EC2:

Should have Public IPv4 address

Private-EC2:

Should NOT have Public IPv4 address
-----------------------------------------------------------------------------------------------------------
8. Test the Setup
Test 1: Public EC2 Internet Access

SSH into Public-EC2:

ssh -i keypair.pem ec2-user@<Public-IP>

Test internet:

ping google.com

If it replies → Public subnet works.

Test 2: Private EC2 No Direct Access

Try SSH from your laptop to Private-EC2.

It should FAIL because:

No public IP

Private subnet

Test 3: SSH to Private EC2 via Public EC2 (Optional Advanced Test)

From Public-EC2:

ssh ec2-user@<Private-EC2-Private-IP>

If it connects → internal VPC communication works.

Final Expected Result

Public EC2 → Internet access

Private EC2 → No direct public access

Both communicate internally inside VPC
