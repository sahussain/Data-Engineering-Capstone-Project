# Data Engineering Nanodegree
## Capstone Project
Ashraf Hussain 
August 3, 2020

Data Engineer Nanodegree: 

## I. Definition

### Project Overview
On 31 December, 2019, the World Health Organization (WHO) was informed of an outbreak of “pneumonia of unknown cause” detected in Wuhan City, Hubei Province, China. Initially identified as coronavirus disease 2019, it quickly came to be known widely as COVID-19 and has resulted in an ongoing global pandemic. As of 5 Augest, 2020, more than 18.95 million cases have been reported across 188 countries and territories, resulting in more than 0.7 million deaths. More than 12.14 million people have recovered.[^1]

In response to this ongoing public health emergency, Johns Hopkins University (JHU), a private research university in Maryland, USA, developed an interactive web-based dashboard hosted by their Center for Systems Science and Engineering (CSSE). The dashboard visualizes and tracks reported cases in real-time, illustrating the location and number of confirmed COVID-19 cases, deaths and recoveries for all affected countries. It is used by researchers, public health authorities, news agencies and the general public.

In that project we will desgine a ETL procress to import csv files form [csse_covid_19_daily_reports_us](https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data/csse_covid_19_daily_reports_us) by JHU which enriched by mapping USA Hospital Beds data from [USA Hospital Beds - COVID-19 | Definitive Healthcare](https://console.aws.amazon.com/dataexchange/home?region=us-east-1#/subscriptions/prod-ydzs6f2cju6qc). The fact table we will use [UID_ISO_FIPS_LookUp_Table.csv](https://github.com/CSSEGISandData/COVID-19/blob/master/csse_covid_19_data/UID_ISO_FIPS_LookUp_Table.csv "UID_ISO_FIPS_LookUp_Table.csv")

This project will tackel the ETL procress using the following tools:
 - [AWS CloudFormation](https://aws.amazon.com/cloudformation/): allows you to use programming languages or a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts. This gives you a single source of truth for your AWS and third party resources.
 - [Apache Airflow](https://airflow.apache.org/): Platform created by the community to programmatically author, schedule and monitor workflows.
 - [Amazon Redshift](https://aws.amazon.com/redshift/): The most popular and fastest cloud data warehouse
 - [AWS Data Exchange](https://aws.amazon.com/data-exchange/): makes it easy to find, subscribe to, and use third-party data in the cloud. Qualified data providers include category-leading brands such as Reuters, who curate data from over 2.2 million unique news stories per year in multiple languages; Change Healthcare, who process and anonymize more than 14 billion healthcare transactions and $1 trillion in claims annually; Dun & Bradstreet, who maintain a database of more than 330 million global business records; and Foursquare, whose location data is derived from 220 million unique consumers and includes more than 60 million global commercial venues

Hear are the summary of the steps we will take to ETL the data into redshift:

- AWS CloudFormation to setup our infrastructure.  
- AWS CloudFormation to bring in data from [JHU COVID-19 git repo](https://github.com/CSSEGISandData/COVID-19)
- AWS Data Exchange to subcribe to [USA Hospital Beds - COVID-19 | Definitive Healthcare](https://console.aws.amazon.com/dataexchange/home?region=us-east-1#/subscriptions/prod-ydzs6f2cju6qc).
- We will use Apache Airflow DAGs to select data for US only, marge the data into one table and move data from S3 to redshift


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

#### Set up the infrastructure
This procress could take up

##### Setup Redshift
1. Download [Data-Engineering-Capstone-Project-Redshift.yaml](https://github.com/sahussain/Data-Engineering-Capstone-Project/blob/master/Infrastructure/Data-Engineering-Capstone-Project-Redshift.yaml "Data-Engineering-Capstone-Project-Redshift.yaml")
2. Go to AWS Cloud Formation page
3. Click Create stack
4. Click upload file and choose [Data-Engineering-Capstone-Project-Redshift.yaml](https://github.com/sahussain/Data-Engineering-Capstone-Project/blob/master/Infrastructure/Data-Engineering-Capstone-Project-Redshift.yaml "Data-Engineering-Capstone-Project-Redshift.yaml")
5. Fill in the stack name as `redshift`
6. Fill in the `Parameters` and click `Next`
7. Don't fill any thing on `Configure stack options` and Click `Next`
8. On Review page click `Create stack`

##### Setup Airflow
```
The stack resources take around 15 minutes to create, while the airflow installation and bootstrap 
another 3 to 5 minutes. After that you can already access the Airflow UI and deploy your own Airflow DAGs.
```
1. Go to [Turbine git repo](https://github.com/villasv/aws-airflow-stack)
2. Scroll down until you see 'Deploy the stack` and click Launch stack
3. On the Create stack page click `Next`
4. Name your stack airflow and click `Next`. You don't need to change anything on this page
5. Don't fill any thing on `Configure stack options` and Click `Next`
6. On Review page Scroll down to `Capabilities` and check both 
	- I acknowledge that AWS CloudFormation might create IAM resources with custom names.
	- I acknowledge that AWS CloudFormation might require the following capability: CAPABILITY_AUTO_EXPAND
7. Click `Create stack`

#### Update Airflow security group to accept HTTP:
 1. Go to EC2 Dashboard
 2. Click Running instances
 3. Find 
 4. Select it the Instance with the name `turbine-webserver`
 5. Under Security Group select the group which contains the name `webserver`
 6. Click on Security group ID for the `webserver`
 7. Under inbound rules
 8. Click edit Edit inbound rules
 9. Click Add rule
 10. From the first(Type) Drop-down select Custom TCP
 11. In the port Range type 8080
 12. From the next drop-down(source) select my IP
 13. Click Save rules

#### Accessing Airflow:
1. Go to EC2 Dashboard
2.  Click Running instances
3. Find 
4. Select it the Instance with the name `turbine-webserver`
5. copy the `Public DNS (IPv4)`
6. open a browser window
7. past the link and add `:8080`

#### Uploading Dags
On Raspberry Pi
1. Create a airflow folder in your home directory you can just use 
2. In terminal Clone git repo [turbine](https://github.com/villasv/aws-airflow-stack) by running `git clone https://github.com/villasv/aws-airflow-stack.git`
3. Copy airflow content form git repo to your airflow folder
4. Copy makefile to home folder
5. go to terminal and run the following command from your home dir 'make deploy stack-name=airflow'
6. if you get an error code 255 make sure that your aws config under .aws which is in your home folder only have the following line 
```
	[default]
	region = us-west-2
```
#### Add AWS credentials to Airflow Connections
use Airflow's UI to configure your AWS credentials and connection to Redshift.
1. To go to the Airflow UI
2. Click on the **Admin** tab and select **Connections**.
3. Under **Connections**, select **Create**.
4. On the create connection page, enter the following values:
	-   **Conn Id**: Enter  `aws_credentials`.
	-   **Conn Type**: Enter  `Amazon Web Services`.
	-   **Login**: Enter your  **Access key ID**  from the IAM User credentials you downloaded earlier.
	-   **Password**: Enter your  **Secret access key**  from the IAM User credentials you downloaded earlier.

Once you've entered these values, select  **Save**.

#### Add AWS redshift to Airflow Connections
Getting redshift connection settings:
1. Go to AWS CloudFormation console
2. click on redshift click on outputs
	here you'll find all the settings for redshift
3. To go to the Airflow UI
4. Click on the **Admin** tab and select **Connections**.
5. Under **Connections**, select **Create**.
6. On the create connection page, enter the following values:
	-   **Conn Id**: Enter  `redshift`.
	-   **Conn Type**: Enter  `Postgres`.
	-   **Host**: Enter the endpoint of your Redshift cluster, excluding the port at the end. **IMPORTANT: Make sure to  NOT**  include the port at the end of the Redshift endpoint string.
	-   **Schema**: Enter  `dev`. This is the Redshift database you want to connect to.
	-   **Login**: Enter  `awsuser`.
	-   **Password**: Enter the password you created when launching your Redshift cluster.
	-   **Port**: Enter  `5439`.

Once you've entered these values, select  **Save**.

### IMPORTANT: Don't forget to shutdown everything. This is very simple
1. Go to AWS CloudFormation console 
2. Select stacks
3. Select `redshift` click `Delete`
4. Select `airflow` click `Delete`
5. The stack will shutdown everything correctly, do not delete Nested stacks or delete es2, s3 buckets independently. It will take up to 15 to 20 minutes to delete. If you don't have any CloudFormation then everything deleted correctly. 
6. Sometimes you will get an error `DELETE_FAILED` in this case you will have to go into `Events` and check why it failed to delete. There is a known error where the current Airflow is not emptying the S3 buckets if they are not empty. The workaround is to run the delete stack once then you get an error then go to S3 and empty the buckets and rerun delete stack. 

## II Database

### Step 1: Scope the Project and Gather Data
The following datasets will be used:
#### [JHU CSSE COVID-19 Dataset](https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data/csse_covid_19_daily_reports_us) 
Provided By: Johns Hopkins University (JHU) via GitRepo

**Data Description**
This table contains an aggregation of each USA State level data.

**File naming convention**
MM-DD-YYYY.csv in UTC.

**Field description**
-   **Province_State**  - The name of the State within the USA.
-   **Country_Region**  - The name of the Country (US).
-   **Last_Update**  - The most recent date the file was pushed.
-   **Lat**  - Latitude.
-   **Long_**  - Longitude.
-   **Confirmed**  - Aggregated case count for the state.
-   **Deaths**  - Aggregated death toll for the state.
-   **Recovered**  - Aggregated Recovered case count for the state.
-   **Active**  - Aggregated confirmed cases that have not been resolved (Active cases = total cases - total recovered - total deaths).
-   **FIPS**  - Federal Information Processing Standards code that uniquely identifies counties within the USA.
-   **Incident_Rate**  - cases per 100,000 persons.
-   **People_Tested**  - Total number of people who have been tested.
-   **People_Hospitalized**  - Total number of people hospitalized.
-   **Mortality_Rate**  - Number recorded deaths * 100/ Number confirmed cases.
-   **UID**  - Unique Identifier for each row entry.
-   **ISO3**  - Officialy assigned country code identifiers.
-   **Testing_Rate**  - Total test results per 100,000 persons. The "total test results" are equal to "Total test results (Positive + Negative)" from  [COVID Tracking Project](https://covidtracking.com/).
-   **Hospitalization_Rate**  - US Hospitalization Rate (%): = Total number hospitalized / Number cases. The "Total number hospitalized" is the "Hospitalized – Cumulative" count from  [COVID Tracking Project](https://covidtracking.com/). The "hospitalization rate" and "Total number hospitalized" is only presented for those states which provide cumulative hospital data.

**Update frequency**
-   Once per day between 04:45 and 05:15 UTC.

**Data sources**
Refer to the  [mainpage](https://github.com/CSSEGISandData/COVID-19).

#### [USA Hospital Beds - COVID-19 | Definitive Healthcare](https://aws.amazon.com/marketplace/pp/USA-Hospital-Beds-COVID-19-Definitive-Healthcare/prodview-yivxd2owkloha)

Provided By: [https://www.definitivehc.com/](https://www.definitivehc.com/)

**Data Description**
This resource includes data on numbers of licensed beds, staffed beds, ICU beds, and the bed utilization rate for the hospitals in the United States.


The end case is to have the data avaliable in Redshift so that we can build on my other project [Udacity-MLEN-CapstoneProject](https://github.com/sahussain/Udacity-MLEN-CapstoneProject) to forecast number of people infected (new and total case), Bed Utilization Rate caused by COVID-19 for a time duration of 30-days in united states based on historical data from JHU.


### Step 2: Explore and Assess the Data
### Step 3: Define the Data Model
### Step 4: Run ETL to Model the Data
### Step 5: Complete Project Write Up

## III. Conclusion





## Addressing other scenarios

### Data Increased by 100x:


### The pipelines would be run on a daily basis by 7 am every day:


### The database needed to be accessed by 100+ people:


------
# Appendix A
[^1]:[COVID-19 Dashboard](https://systems.jhu.edu/research/public-health/ncov/) by the Center for Systems Science and Engineering (CSSE) at Johns Hopkins University (JHU)". ArcGIS. Johns Hopkins University. Retrieved 20 June 2020.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEzMjkzODAzMjEsNzMwODA3MjY5LC0xOT
cwNDIwNTk4LDk2MTAwNzkwMywxODg2ODIyMTM5LDE4ODU4MTU3
NjgsMTE2Nzg0MTg1LDE0NTA2ODYyODgsMTY5Nzk1NDY4NywtMj
UwNjIxNTgzLDIwODk0MTY3NDYsLTE4NjM5NzQ5ODcsMTAwNjE4
MjM2MywtMTYxOTE0MjE1OCwtMTM0NzcxNjI1NCwxNzUzOTMyND
QzLC0xODkwMDAwNjE0LDQzNjI3NzA3LC00MDQyMzY5OTAsLTE1
MzM4NjYxMzVdfQ==
-->