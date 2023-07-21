# AWS Cloud & Big Data Architectures Capstone Project
Maxime LARROZE <br>
Wilfried Jordan FOKANA <br>
Thomas MIRAS GARCIA <br> <br>
---

Table of Contents

[1. AWS Cloud - The Website](#awscloud) <br>
[2. Quizz IAM & Network](#quizz) <br>
[3. IAM policies](#iam) <br>
[4. Big Data - Data Visualization With AWS QuickSight](#bigdata) <br>

---
<div id='awscloud'/>
  
# 1. AWS Cloud - The Website

![image](AWSCloudWebsite/Cloud_Architecture_Project.png) <br><br>

1| First we created a VPC, with the the IPv4 to 10.0.0.0/24 : <br>
![image](AWSCloudWebsite/wb1.png) <br><br>

2| Then we added a public subnet to our VPC, with the IPv4 to 10.0.0.0/25 (with settings enabling auto-assign public IPv4 address) : <br>
![image](AWSCloudWebsite/wb11.png)<br><br>

3| And we needed an internet gateway, to attach it to our VPC : <br>
![image](AWSCloudWebsite/wb20.png)<br><br>

4| We updated the VPC route tables, by adding a new route (0.0.0.0/0) to the internet gateway from anywhere : <br>
![image](AWSCloudWebsite/wb3.png) <br><br>

5| Finally, we created an endpoint in the VPC for it connect to the EC2 instance remotely : <br>
![image](AWSCloudWebsite/wb5.png) <br><br>

6| Now we created an instance based on the AMI **Cloud9AmazonLinux2-2023-06-22T17-21.** : <br>
![image](AWSCloudWebsite/wb2.png) <br><br>

7| We did the required installation that was given in the subject, by connecting to the instance in the terminal : <br>
<pre>
  sudo yum -y update
  sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
  sudo yum install -y httpd mariadb-server
  sudo chkconfig httpd on
  sudo service httpd start
  cd /home/ec2-user/environment
  wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-2/21-course-project/s3/Countrydatadump.sql
  ln -s /var/www/ /home/ec2-user/environment/
  sudo chown -R ec2-user:ec2-user Countrydatadump.sql
  cd /var/www/html
  wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-2/21-course-project/s3/Example.zip
  sudo unzip Example.zip -d /var/www/html/
  sudo chown -R ec2-user:ec2-user /var/www/html
</pre>
![image](AWSCloudWebsite/wb8.png)
![image](AWSCloudWebsite/wb7.png)<br><br>

8| We also added the required inbound rules in the security group (Launch-Wizard-5) of our instance (HTTP with port 80) : <br>
![image](AWSCloudWebsite/wb6.png) <br><br>

9| Finally it worked the website was accessible on the public IP provided by our instance **http:// 13.51.79.121** (step 6, instance details)
![image](AWSCloudWebsite/wb9.png) <br><br>

10| Now we want to query on the website with the content of a database, as such we created one with MariaDB on RDS : <br>
![image](AWSCloudWebsite/wb10.png) <br>
No need to create a private subnet, we did it before but we saw it was useless afterwards. <br>
It is done automatically by RDS but for the public one we used the one we created earlier (**ID 01a5b486b3e232222**) <br><br>

11| As our VPC & EC2 instance are linked to the database, <br>
we can just connect to MariaDB on the terminal to upload the content of **Countrydatadump.sql** :
![image](AWSCloudWebsite/wb12.png)
![image](AWSCloudWebsite/wb13.png) <br><br>

12| We created parameters in the parameter store to store the database name, endpoint, username and password : <br>
![image](AWSCloudWebsite/wb17.png) <br><br>

13| Now our EC2 instance needs to read our RDS database parameters for it to access its content <br>
As such we first created a policy to allow the EC2 instance to connect to the database :<br>
![image](AWSCloudWebsite/wb14.png) <br><br>

14| For the EC2 instance to use this policy we created a role: <br>
![image](AWSCloudWebsite/wb15.png) <br><br>

15| We attached this role to our EC2 instance : <br>
![image](AWSCloudWebsite/wb16.png) <br><br>

16| And finally we can go back to our website and try the queries and see that it works ! <br>
![image](AWSCloudWebsite/wb18.png)
![image](AWSCloudWebsite/wb19.png) 
<br><br>

---
<div id='quizz'/>
  
# 2. Quizz IAM & Network

## IAM quizz
1. Option 3 
2. Option 1
3. Option 3 & 4
4. Option 1
5. Option 1
6. Option 3
7. Option 1

## Network quizz
1. Option 3 
2. Option 3
3. Option 1, 3 & 6
4. Option 3 
<br><br>

---
<div id='iam'/>
  
# 3. IAM policies

**Question 1: What actions are allowed for EC2 instances and S3 objects based on this policy? What specific resources are included?** <br>
Based on the provided policy, the following actions are allowed :<br>
For EC2 instances : <br>
``ec2:RunInstances:`` Allows the user to launch new EC2 instances. <br>
``ec2:TerminateInstances:`` Allows the user to terminate EC2 instances. <br>
For S3 objects : <br>
``s3:GetObject:`` Allows the user to retrieve (read) objects from an S3 bucket.
``s3:PutObject:`` Allows the user to upload (write) objects to an S3 bucket.
<br><br>
The policy includes the following specific resources:<br>
For EC2 instances :<br>
``arn:aws:ec2:us-east-1:123456789012:instance/*`` : This specifies all EC2 instances in the us-east-1 region under the AWS account 123456789012. The * indicates that the policy applies to all instances under that account and region. 
<br>
For S3 instances : <br>
``arn:aws:s3:::example-bucket/*`` : This specifies all objects within the S3 bucket named example-bucket. The * indicates that the policy applies to all objects within that bucket.
<br><br>

**Question 2: Under what condition does this policy allow access to VPC-related information? Which AWS region is specified?** <br>
This policy allows access to VPC-related information under the condition that the aws:RequestedRegion value is equal to "us-west-2". The specified AWS region is "us-west-2".
<br><br>

**Question 3: What actions are allowed on the "example-bucket" and its objects based on this policy? What specific prefixes are specified in the condition?** <br>
Based on this policy, the following actions are allowed on the "example-bucket" and its objects: <br>
``s3:GetObject:`` Allows retrieving (reading) objects from the "example-bucket". <br>
``s3:PutObject:`` Allows uploading (writing) objects to the "example-bucket". <br>
``s3:ListBucket:`` Allows listing the objects within the "example-bucket". <br>

The policy specifies the following prefixes in the condition: <br>
``documents/*:`` This prefix specifies that objects within the "example-bucket" under the "documents" directory or folder are included in the allowed actions. For example, objects with keys like "documents/file1.txt" or "documents/subfolder/file2.txt" would match this prefix. <br>
``images/*:`` This prefix specifies that objects within the "example-bucket" under the "images" directory or folder are included in the allowed actions. For example, objects with keys like "images/photo.jpg" or "images/subfolder/image.png" would match this prefix. <br>
The policy allows the specified actions on any object within the "example-bucket" that matches either the "documents/" or "images/" prefix.
<br><br>

**Question 4: What actions are allowed for IAM users based on this policy? How are the resource ARNs constructed?** <br>
Based on this policy, the following actions are allowed for IAM users: <br>
``iam:CreateUser:**`` Allows creating new IAM users. <br>
``iam:DeleteUser:**`` Allows deleting existing IAM users. <br>

The resource ARNs in this policy are constructed using the ${aws:username} variable. 
The ${aws:username} variable is a placeholder that will be dynamically replaced with the actual username of the IAM user during the evaluation of the policy. 
It allows the policy to apply to the specific IAM user who is making the request. <br>
The resource ARN for both statements in the policy is ``arn:aws:iam::2546542454567:user/${aws:username}`` <br>
In this ARN, 2546542454567 represents the AWS account ID, and ${aws:username} is the placeholder for the IAM user's username.
<br><br>

**Questions 5-1: Which AWS service does this policy grant you access to?** <br>
This policy grants access to the AWS Identity and Access Management (IAM) service.
<br><br>

**Questions 5-2: Which AWS service does this policy grant you access to?** <br>
While this policy allows various iam:Get* and iam:List* actions, it does not explicitly grant permissions to create an IAM user, group, policy, or role. <br>
The actions specified in this policy are focused on retrieving (getting) and listing IAM-related information and resources, such as users, groups, policies, roles, access keys, and more. To create an IAM user, group, policy, or role, additional iam:Create* or specific actions for creation would need to be included in the Action field of the policy statement.
<br><br>

**Questions 5-3: Name at least three specific actions that the iam:Get action allows.** <br>
``iam:GetUser:`` Grants permission to retrieve information about the specified IAM user, including user's creation date, path, unique ID, and ARN. <br>
``iam:GetRole:`` Grants permission to retrieve information about the specified role, including role's path, GUID, ARN, and the role's trust policy. <br>
``iam:GetPolicy:`` Grants permission to retrieve information about the specified managed policy, including the policy's default version. <br>
``iam:GetGroup:`` Grants permision to the user to retrieve information about a specific IAM group, including its name, path, unique ID, and ARN. 
<br><br>

**Questions 6-1: Which AWS service does this policy grant you access to?** <br>
Based on this policy, the policy allows the following actions: <br>
``ec2:RunInstances:`` Allows launching new EC2 instances. <br>
``ec2:StartInstances:`` Allows starting existing EC2 instances. <br>
However, there is a Deny effect specified in the policy, which means that these actions are denied under certain conditions. The policy denies the ec2:RunInstances and ec2:StartInstances actions when the ec2:InstanceType condition matches "t2.micro" or "t2.small". <br>
This means that the policy is preventing the launch and start of EC2 instances with the instance types of t2.micro or t2.small.
<br><br>

**Questions 6-2: How would the policy restrict the access granted to you by this additional statement?** <br>
The additional statement with the following configuration, will override the previous Deny effect for the ec2:* actions : <br>
<pre>{
    "Effect": "Allow",
    "Action": "ec2:*"
}</pre>
This means that the ec2:* actions, which encompass all EC2 actions, will be allowed without any restrictions. As a result, the Deny effect specified in the previous statement will not restrict access to any EC2 actions.
<br><br>

**Questions 6-3: If the policy included both the statement on the left and the statement in question 2, could you terminate an m3.xlarge instance that existed in the account?** <br>
The provided policy allows all actions on all EC2 instances without imposing any restrictions on terminating instances. Therefore, you would be able to terminate an m3.xlarge instance if it exists in the account.
<br><br>

---
<div id='bigdata'/>
  
# 4. Big Data - Data Visualization With AWS QuickSight

Global view <br>
![image](BigDataQuickSight/qs17.png) <br><br>

Sum of Revenue by Admit Date <br>
![image](BigDataQuickSight/qs1.png) <br><br>

Sum of Profit by Hospital <br>
![image](BigDataQuickSight/qs2.png)

Sum of Cost per Category and Admit Date <br>
![image](BigDataQuickSight/qs3.png)
![image](BigDataQuickSight/qs4.png)
![image](BigDataQuickSight/qs5.png)
![image](BigDataQuickSight/qs6.png) <br><br>

Sum of Discount and Sum of Profit by Admit Date <br>
![image](BigDataQuickSight/qs7.png)
![image](BigDataQuickSight/qs8.png) <br><br>

Sum of Revenue by Admit Date <br>
![image](BigDataQuickSight/qs9.png)
![image](BigDataQuickSight/qs10.png) <br><br>

Sum of Profit by Admit Date <br>
![image](BigDataQuickSight/qs11.png)
![image](BigDataQuickSight/qs12.png) <br><br>

Period over Period <br>
![image](BigDataQuickSight/qs13.png) 
![image](BigDataQuickSight/qs14.png) <br><br>

Top 3 Category for total Profit are : <br>
![image](BigDataQuickSight/qs15.png)
![image](BigDataQuickSight/qs16.png) <br><br>
