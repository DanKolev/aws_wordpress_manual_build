# Create a single server for web app

![Server](https://github.com/DanKolev/aws_wordpress_manual_build/blob/main/diagrams/2.setup_wordpress_server_manually.png)

# Overview

 - Create a security group for the EC2 instance.

 - Setup the environment which WordPress will run from.

 - Configure `Session Manager` as means to access the server.

 - Configure SSM Parameters which the manual and automatic stages will use and perform a manual install of wordpress and a database on the same EC2 instance.

# Create security group for the EC2 instance
Click `Create Security Group`

For name use `VPCSPCSG`

For `Description` type `Allow HTTPS IN`

Under `Inbound Rules` click `Add rule`

Under `Type` enter and select HTTPS, and a CIDR block of 0.0.0.0/0

Click on `Create Security Group`

# Configure Session Manager login

Under `IAM` select `Role`, then click on `Create Role`

For `Trusted entity type` ensure that `AWS service` is selected.

Under `Common use cases` select `EC2`

Under `Add permissions` paste the policy name `AmazonSSMManagedInstanceCore`, and click `Next`.

Under `Name, Review and Create`, scroll all the way don and click on `Create Role`.

# Create Instance to run Wordpress

Click `Launch Instance`

For name use `Wordpress-Manual`

Select `Amazon Linux`

From the dropdown make sure `Amazon Linux 2 AMI (HVM), SSD Volume Type' AMI is selected ensure `64-bit (x86)` is selected in the architecture dropdown.

Under instance `type` we will use t3.micro, as it is free.

Under `Key Pair(login)` select `Proceed without a KeyPair (not recommended)`

For `Network Settings`, click `Edit` and in the VPC download select $vpcname

For `Subnet` select `sn-Web-A`

Make sure for both `Auto-assign public IP` and `Auto-assign IPv6 IP` you set to `Enable`

Under security Group Check `Select an existing security group`

Select `SPCVPC-SGWordpress` it will have randomness after it, thats ok :)

We will leave storage as default so make no changes here

Expand `Advanced Details`

For IAM `instance profile role` select `SPCVPC-WordpressInstanceProfile`

Find the `Credit Specification Dropdown` and choose `Standard` (some accounts aren't enabled for Unlimited) Click `Launch Instance`

Click `View All instances`

Wait for the instance to be in a RUNNING state

# Create SSM Parameter Store values for wordpress

SSM Parameter Store will be used to store configuration information. 

Create Parameter - DBUser (the login for the specific wordpress DB)

 - Click `Create Parameter` Set Name to `/SPC/Wordpress/DBUser` Set Description to `Wordpress Database User`

 - Set Tier to `Standard`

 - Set Type to `String`

 - Set Data type to `text`

 - Set Value to `spcwordpressuser`

 - Click `Create parameter`

Create Parameter - DBName (the name of the wordpress database)

Click `Create Parameter` Set Name to `/SPC/Wordpress/DBName` Set Description to `Wordpress Database Name`

 - Set Tier to `Standard`

 - Set Type to `String`

 - Set Data type to `text`

 - Set Value to `spcwordpressdb`

 - Click `Create parameter`

Create Parameter - DBEndpoint (the endpoint for the wordpress DB .. )

 - Click `Create Parameter` Set Name to `/SPC/Wordpress/DBEndpoint` Set Description to `Wordpress Endpoint Name`

 - Set Tier to `Standard`

 - Set Type to `String`

 - Set Data type to `text`

 - Set Value to `localhost`

 - Click `Create parameter`

Create Parameter - DBPassword (the password for the DBUser)

 - Click `Create Parameter` Set Name to `/SPC/Wordpress/DBPassword` Set Description to `Wordpress DB Password`

 - Set Tier to `Standard`

 - Set Type to `SecureString`

 - Set `KMS Key Source` to `My Current Account`

 - Leave KMS Key ID as default Set Value to `<password>` Click `Create parameter`

Create Parameter - DBRootPassword (the password for the database root user, used for self-managed admin)

 - Click `Create Parameter` Set Name to /SPC/Wordpress/DBRootPassword Set Description to `Wordpress DBRoot Password`

 - Set Tier to `Standard`

 - Set Type to `SecureString`

 - Set `KMS Key Source` to `My Current Account`

 - Leave KMS Key ID as default Set Value to `<password>` Click `Create parameter`

# Connect to the instance and install a database and wordpress

 - Right click on `Wordpress-Manual` choose `Connect` choose `Session Manager`
 - Click `Connect`
 - Type `sudo bash` and press Enter
 - Type `cd` and press Enter
 - Type `clear` and press Enter

Bring in the parameter values from SSM

Run the following commands to bring the parameter store values into ENV variables to make the manual build easier.

```html
DBPassword=$(aws ssm get-parameters --region us-east-1 --names /SPC/Wordpress/DBPassword --with-decryption --query Parameters[0].Value)
DBPassword=`echo $DBPassword | sed -e 's/^"//' -e 's/"$//'`

DBRootPassword=$(aws ssm get-parameters --region us-east-1 --names /SPC/Wordpress/DBRootPassword --with-decryption --query Parameters[0].Value)
DBRootPassword=`echo $DBRootPassword | sed -e 's/^"//' -e 's/"$//'`

DBUser=$(aws ssm get-parameters --region us-east-1 --names /SPC/Wordpress/DBUser --query Parameters[0].Value)
DBUser=`echo $DBUser | sed -e 's/^"//' -e 's/"$//'`

DBName=$(aws ssm get-parameters --region us-east-1 --names /SPC/Wordpress/DBName --query Parameters[0].Value)
DBName=`echo $DBName | sed -e 's/^"//' -e 's/"$//'`

DBEndpoint=$(aws ssm get-parameters --region us-east-1 --names /SPC/Wordpress/DBEndpoint --query Parameters[0].Value)
DBEndpoint=`echo $DBEndpoint | sed -e 's/^"//' -e 's/"$//'`
```

Install updates

```html
sudo yum -y update
sudo yum -y upgrade

Install Pre-Reqs and Web Server

sudo yum install -y mariadb-server httpd wget
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
sudo amazon-linux-extras install epel -y
sudo yum install stress -y
```

Set DB and HTTP Server to running and start by default

```html
sudo systemctl enable httpd
sudo systemctl enable mariadb
sudo systemctl start httpd
sudo systemctl start mariadb
```

Set the MariaDB Root Password

```html
sudo mysqladmin -u root password $DBRootPassword
```

Download and extract Wordpress

```html
sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
cd /var/www/html
sudo tar -zxvf latest.tar.gz
sudo cp -rvf wordpress/* .
sudo rm -R wordpress
sudo rm latest.tar.gz
```

Configure the wordpress wp-config.php file

```html
sudo cp ./wp-config-sample.php ./wp-config.php
sudo sed -i "s/'database_name_here'/'$DBName'/g" wp-config.php
sudo sed -i "s/'username_here'/'$DBUser'/g" wp-config.php
sudo sed -i "s/'password_here'/'$DBPassword'/g" wp-config.php
```

Fix Permissions on the filesystem

```html
sudo usermod -a -G apache ec2-user   
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www
sudo find /var/www -type d -exec chmod 2775 {} \;
sudo find /var/www -type f -exec chmod 0664 {} \;
```

Create Wordpress User, set its password, create the database and configure permissions

```html
sudo echo "CREATE DATABASE $DBName;" >> /tmp/db.setup
sudo echo "CREATE USER '$DBUser'@'localhost' IDENTIFIED BY '$DBPassword';" >> /tmp/db.setup
sudo echo "GRANT ALL ON $DBName.* TO '$DBUser'@'localhost';" >> /tmp/db.setup
sudo echo "FLUSH PRIVILEGES;" >> /tmp/db.setup
sudo mysql -u root --password=$DBRootPassword < /tmp/db.setup
sudo rm /tmp/db.setup
```

Test Wordpress is installed

 - Open the EC2 console

 - Select the `Wordpress-Manual` instance

 - Copy the IPv4 Public IP into your clipboard (DON'T CLICK THE OPEN LINK ... just copy the IP) Open that IP in a new tab

 - You should see the wordpress welcome page

Perform Initial Configuration and make a post

 - In `Site Title` enter `Planetarium`

 - In `Username` enter `admin` in `Password` it should suggest a strong password for the wordpress admin user, feel free to use this or choose your own. Write it down somewhere safe.

 - In `Your Email` enter your email address

 - Click `Install WordPress` click `Log In`

 - In `Username or Email Address` enter `admin`

 - In `Password` enter the previously noted down strong password click `Log In`


 - Click `Posts` in the menu on the left
 
 - Select `Hello World!` click `Bulk Actions` and select `Move to Trash` click `Apply`

 - Click `Add New`

 - If you see any popups close them down

 - For title `Space is Incredible!`

 - Click the `+` under the title, select `Gallery` click `Upload`

 - Select some pictures of planets.

 - Upload them

 - Click `Publish`

 - Click `View Post`

This is your working, manually installed and configured wordpress.

This configuration has several limitations which you will be resolved one by one :-

    The application and database are built manually, taking time and not allowing automation
    ^^ it was slow and annoying ... that was the intention.
    The database and application are on the same instance, neither can scale without the other
    The database of the application is on an instance, scaling IN/OUT risks this media
    The application media and UI store is local to an instance, scaling IN/OUT risks this media
    Customer Connections are to an instance directly ... no health checks/auto healing
    The IP of the instance is hardcoded into the database ....
    
    Right click Wordpress-Manual , Instance State, Stop, Yes, Stop
    Right click Wordpress-Manual , Instance State, Start, Yes, Start
    the IP address has changed ... which is bad
    Try browsing to it ...
    What about the images....?
    The images are pointing at the old IP address...
    Right click Wordpress-Manual , Instance State, Terminate, Yes, Terminate



