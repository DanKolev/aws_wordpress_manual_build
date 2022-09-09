# Overview

This project outlines building a fully automated, three-tier (db,app,public) environment. At the beginning, the application and database are built manually, 
taking time and not allowing automation. This process is automated in part 3 by using a lauch template.

The database and application are on the same instance, neither can scale without the other. The database of the application is on an instance, 
scaling IN/OUT risks this media. By splitting out the database functionality from the EC2 instance, and running MariaDB to an RDS instance running 
MySQL Engine will allow the DB and Instance to scale independently, and will allow the data to be secure past the lifetime of the EC2 instance. 

The application media and UI store is local to an instance, scaling IN/OUT risks this media. In part 5, we are creating an EFS file system 
designed to store the wordpress locally stored media. This area stores any media for posts uploaded when creating the post as well as theme data.  
By storing this on a shared file system it means that the data can be used across all instances in a consistent way, and it lives on past the lifetime 
of the instance.

Customer Connections are to an instance directly. There are no health checks, nor auto healing. The IP of the instance is hardcoded into the database.
This is fixed in part 6, by adding an auto scaling group to provision and terminate instances automatically based on load on the system. Which also enabled us
to create a new Paramater store value the elastic balancer's DNS name.

In Part 7, we are upgrading RDS to Aurora, which provides more durability, scalability, resiliency, and performance when compared to RDS.

Thank you!





