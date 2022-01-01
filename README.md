# HIGH-AVAILABILITY-ARCHITECTURE-WITH-AWS-CLI

CONFIGURING WEB-SERVER ON TOP OF AWS EC2-INSTANCE USING S3 BUCKET AND CLOUD FRONT SERVICES OF AWS

Welcome you all, in this blog we will see “HOW WE CAN CREATE HIGH AVAILABILITY ARCHITECTURE WITH AWS CLI”.


TASK DESCRIPTION:-
Create High Availability Architecture with AWS CLI :-
The architecture includes-
- Webserver configured on EC2 Instance.
- Document Root(/var/www/html) made
persistent by mounting on EBS Block Device.
- Static objects used in code such as
pictures stored in S3.
- Setting up Content Delivery Network using
CloudFront and using the origin domain as S3 bucket.
- Finally place the Cloud Front URL on the
webapp code for security and low latency.
Let’s start:-

STEP-1:- First install AWS CLI on your system. For working in the AWS CLI we have to first create one IAM user of our AWS account, this user will have their Access and Secret Key, using this keys we can log-in inside the our aws account from CLI. Command to log-in :-
aws configure


Here it will ask you first to provide Access Key ID of the user, then Secret Key ID, then region name means in which region(Mumbai-ap-south-1, etc) of AWS you wanna work, and then at last the output format means in which form you wanna see your command output(JSON, etc).

STEP-2:- We are successfully log-in inside the our account using the IAM user. Now our first step is to launch the EC2-instance in the AWS. Using “aws help” command we can see all the services of the AWS.
AWS EC2:- Amazon Elastic Compute Cloud (EC2) is a part of Amazon.com’s cloud-computing platform, Amazon Web Services (AWS), that allows users to rent virtual computers on which to run their own computer applications.
#Command to launch the ec2-instance:-
aws ec2 run-instances — image-id INSTANCE_IMAGE_ID — instance-type TYPE_OF_YOUR_INSTANCE — count NUMBER_OF_INSTANCE — key-name KEY_PAIR_NAME_OF_AWS


Using “aws ec2 run-instances help” command we can see all the possible properties that we can include in the command. Here I’m using Amazon Linux 2 instance Image ID.
Our instance is successfully launched, we can see it in AWS Console-


STEP-3:- Now we have to create one Block Storage(EBS).
AWS EBS:- Amazon Elastic Block Store (EBS) is an easy to use, high performance block storage service designed for use with Amazon Elastic Compute Cloud (EC2) for both throughput and transaction intensive workloads at any scale.
Command to create the EBS Volume:-
aws ec2 create-volume — size SIZE_OF_THE_VOLUME — availability-zone AZ_OF_AWS.
NOTE:- WE HAVE TO CREATE THIS VOLUME ONLY IN THAT AZ IN WHICH MY INSTANCE IS RUNNING.


Here I’m creating volume of 10GIB in size and in the ap-south-1a AZ. Using “aws ec2 create-volume help” command we will find all the possible properties that we can include in this command.
Volume is successfully created-


STEP-4:- Now we have to attach this EBS volume to our EC2-instance for using it.
Command to attach:-
aws ec2 attach-volume — instance-id EC2_INSTANCE_ID — device NAME_THE_DEVICE(“/dev/sdf, /dev/xvdh, etc”) — volume-id EBS_VOLUME_ID
Using this command we successfully attached our EBS-Volume to our instance. We can see it by going inside the Instance either from PUTTY or directly from AWS EC2 Console if connect option is present.
Here we see that now our volume is in IN-USE state.




Here you can see the new volume is connected with 10GB of size.

STEP-5:- We have to configure the web-server on top of this instance, so if you are using Linux OS then using “yum” we can install the web-server. Command:
yum install httpd


Finally our web-server is installed.

STEP-6:- Now For using any of the Block Storage, first we have to create partition of that storage, then have to format it and then we have to mount that storage to any of the directory of our instance. So let’s first create partition of this storage or volume.
Inside the instance run this commands:-
1- sudo su -root:- for log-in into the root account.
2- fdisk /dev/xvdh:- I have created with the “xvdh” name and then type this commands given in below image:-


This will create partition of our volume.
3- mkfs.ext4 /dev/xvdh1:- For formatting the partition.


4- mount /dev/xvdh1 /var/www/html:- Mounting the partition to the directory /var/www/html, this directory is the main directory of Apache Web-server. It only access those web pages which are present inside this directory.


As we can see our volume is successfully mounted with the given directory.

STEP-7:- Now we have to create the S3 Bucket for storing the static objects(Image, videos, etc) that we gonna use in our web pages.
S3 Bucket:- S3 is a storage service offered by Amazon. It stands for simple storage service and provides cloud storage for various types of web development applications.
Command to create the s3 bucket:-
aws s3api create-bucket — bucket NAME_OF_THE_BUCKET — region AWS_REGION — acl READ_ACCESS(PUBLIC, PRIVATE) — create-bucket-cofiguration LocationConstraint=ap-south-1


Here I’m using taskwebserver as my bucket name and launching it in ap-south-1(mumbai) region with public read access.


Our bucket is successfully created.

STEP-8:- Now we have to copy our Image from our Local computer to the S3 bucket. Command:-
aws s3 cp LOCALPATH BUCKET_PATH — acl READ_ACCESS

IMAGE is successfully copied inside the s3 bucket.

STEP-9:- Now we have to Set-up Content Delivery Network using
CloudFront and using the origin domain as S3 bucket.
CloudFront:- Amazon CloudFront is a content delivery network (CDN) offered by Amazon Web Services. Content delivery networks provide a globally-distributed network of proxy servers which cache content, such as web videos or other bulky media, more locally to consumers, thus improving access speed for downloading the content.
Command to create the cloud front:-
aws cloudfront create-distribution — origin-domain-name S3_BUCKET_DOMAIN_NAME




Our cloudfront distributions is successfully created.
STEP-10:- Now we have to create web page for our webserver inside the /var/www/html directory.
Commands:-
1- sudo su -root:- Root User.
2- cd /var/www/html:- To go inside the directory.
3- vi index.html:- Creating a web page with name index. Here in the image url I’m providing the Cloud Front domain url with my image name at last:


4- systemctl start httpd:- For starting the web-server service.
STEP-11:- Now we know that the web-server runs in http or https protocol with having 80 port number. So we have to add inbound rules in our instance security group for allowing the http request traffic
