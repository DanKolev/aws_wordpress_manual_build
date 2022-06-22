# Create a single server for web app

![subnets](https://github.com/DanKolev/aws_wordpress_manual_build/blob/main/data/part_2_ec2/EC2.png)

# Overview

 - Setup the environment which WordPress will run from.

 - CREATE SECURITY GROUP FOR EC2 MUST BE PUT IN THE NOTES

 - Configure SSM Parameters which the manual and automatic stages will use and perform a manual install of wordpress and a database on the same EC2 instance.

 # Create Instance to run Wordpress

Click Launch Instance

For name use `Wordpress-Manual`

Select `Amazon Linux`

From the dropdown make sure `Amazon Linux 2 AMI (HVM), SSD Volume Type' AMI is selected ensure `64-bit (x86)` is selected in the architecture dropdown.

Under instance `type` we will use t3.micro, as it is free.

Under `Key Pair(login)` select `Proceed without a KeyPair (not recommended)`

For `Network Settings`, click `Edit` and in the VPC download select $vpcname

For `Subnet` select `sn-Web-A`

Make sure for both `Auto-assign public IP` and `Auto-assign IPv6 IP` you set to `Enable`

Under security Group Check `Select an existing security group`

Select `LabVPC-SGWordpress` it will have randomness after it, thats ok :)

We will leave storage as default so make no changes here

Expand `Advanced Details`

For IAM `instance profile role` select `LabVPC-WordpressInstanceProfile`

Find the `Credit Specification Dropdown` and choose `Standard` (some accounts aren't enabled for Unlimited) Click `Launch Instance`

Click `View All instances`

Wait for the instance to be in a RUNNING state



