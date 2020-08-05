# Data Engineering Nanodegree
## Capstone Project
Ashraf Hussain 
August 3, 2020

Data Engineer Nanodegree: 

## I. Definition

### Project Overview
The purpose of the data engineering capstone project is to give a chance to combine what you've learned throughout the program. This project will be an important part of your portfolio that will help you achieve your data engineering-related career goals.

In this project, you can choose to complete the project provided for you, or define the scope and data for a project of your own design. Either way, you'll be expected to go through the same steps outlined below.

### Problem Statement


### Prerequisites

#### IAM user:
 1. Create a new IAM user in your AWS account
 2. Give it `AdministratorAccess`, From `Attach existing policies directly` Tab
 3. Take note of the access key and secret 

#### Install AWS CLI and Configure
 1. Follow the instructions on 
	 - [Installing the AWS CLI version 2 on Windows](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html)
	 - [Configure the AWS CLI on a Raspberry Pi](https://ownthe.cloud/posts/configure-aws-cli-on-raspberry-pi/)
3. Run `aws configure`
4. Past access key and secret 

#### Generate An AWS Key Pair for Airflow
 1. create an Amazon EC2 key pair:
 2. Go to the Amazon EC2 console
 3. In the Navigation pane, click Key Pairs
 4. On the Key Pairs page, click Create Key Pair
 5. In the Create Key Pair dialog box, enter a name for your key pair, such as, mykeypair
 6. Make sure that you select ppk under File format
 7. Click Create
 8. Save the resulting PEM file in a safe location

#### Update eks-cluster-sg-udacityDataEng1-349815526 security group to accept SSH:
 1. Go to EC2
 2. Click Running instances
 3. Find 
 4. Select it
 6. Under Network & Security select Security Group
 7. Select Security group ID with the name ElasticMapReduce-master
 8. Click edit Edit inbound rules
 9. Click Add rule
 10. From the first(Type) Drop-down select SSH
 11. From the next drop-down(source) select my IP
 12. Click Save rules

#### Set up the infrastructure
1. Download [Data-Engineering-Capstone-Project-Redshift.yaml](https://github.com/sahussain/Data-Engineering-Capstone-Project/blob/master/Infrastructure/Data-Engineering-Capstone-Project-Redshift.yaml "Data-Engineering-Capstone-Project-Redshift.yaml")
2. Go to AWS Cloud Formation page
3. Click Create stack
4. Click upload file and choose [Data-Engineering-Capstone-Project-Redshift.yaml](https://github.com/sahussain/Data-Engineering-Capstone-Project/blob/master/Infrastructure/Data-Engineering-Capstone-Project-Redshift.yaml "Data-Engineering-Capstone-Project-Redshift.yaml")
5. Fill in the Parameters and click next
6. Don't fill any thing on Configure stack options and Click next
7. On Review page click `Create stack`

## II Database

###  Schema Design

### ELT Pipeline

### Project Datasets

### Fact Table


### Dimension Tables


## III. Conclusion





## Addressing other scenarios

### Data Increased by 100x:


### The pipelines would be run on a daily basis by 7 am every day:


### The database needed to be accessed by 100+ people:
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxNDAxMzEwNzQsLTQwNDIzNjk5MCwtMT
UzMzg2NjEzNSwtNTQzMDg4ODQ0LDQ3OTEwODM0MywxOTgwNTYx
NDQ1LC01OTg3NjcxNjgsLTE5ODU4NDA0MTMsMTU0NjU3NjM5OC
wxOTc2MjA2NjQxLDEyODU4MDg3ODQsMjAxNTE1ODg3NF19
-->