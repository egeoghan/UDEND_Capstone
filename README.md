# UDEND_Capstone

The data being used for this project is I94 Immigration Data in the SAS format and the Airport data in CSV. We will transform the datasets accordingly to be used in order to gain insights on popular airport and city hubs for immigration in the United States. 

## Goal
The goal of this project is to use immigration and airport data to help a immigration focused non profit succeed. Initially, and on a very tight deadline, they want to see what airports are most popular amongst immigrants to the USA. This data will fuel decisions around how to best assist immigrants in their travels and work within the states to better staff their offices around the major immigration hubs. The project uses spark to make a data lake in S3, as well as load table into redshift. Redshift will create a data warehouse for the analytics team to get what they need regarding immigration airport hubs quickly. The data lake in S3 along with the staging tables is for future analysis, since there was also a requirement from the non profit to keep any valuable data pertaining to the immigration events. For now we will keep the data in the immigrations_data relatively raw, so that we can get what we need regarding the airport hub very quickly. Next steps for the data are discussed below.

For this scenario, we wouldn’t really need data updated too often, probably only annually since it will impact something that can’t be changed all too quickly. However, the other unknown use cases for the data lake may require the data to be updated more often. 

Since the project is slotted to figure out where to post workers for the nonprofit in the first quarter of next year, we will only use the first couple months of data. However, we may want to consider using more as time goes on.


## Tools and Technologies
This project uses Spark to do distributed data processing. I chose to use the Python which is supported by Spark to perform the ETL process in the etl.py file. I chose to use Spark instead of Pandas because it is a technology I am new to and wanted to try it out. I also really like the .alias function used in the Pyspark library. I chose to use S3 because it is a very logical choice for a data lake because of its easy key value storage and handling of parquet files which can easily be queried for any questions. I chose Redshift for the data warehouse as an option so that data scientists on the team would easily be able to plug in with AWS Quicksight to deliver on this time deadline the fastest way possible. 

I chose to use the star schema data model for this application as can be seen below. This enables users to gain quick insights on immigration travel and a closer look at airports and time details as they see fit. Based on the requirements, this model will store the data in the most efficient way so that time and airport details don't need to be replicated for each immigration event. 

### Sample Queries

```
SELECT COUNT(airport_code)
FROM immigrations_table
WHERE immigrations_table.airport_code = 'ATL';
```

```
SELECT COUNT(DISTINCT airport_code)
FROM immigrations_table;
```


Find the most common states 
```
SELECT airports_table.state_code, count(immigrations_table.airport_code) as flight_count
FROM immigrations_table
JOIN airports_table
ON airports_table.airport_code = immigrations_table.airport_code
GROUP BY immigrations_table.airport_code, airports_table.state_code
ORDER BY flight_count DESC
LIMIT 10;
```


Find the states with the most unique airports used 
```
SELECT airports_table.state_code, count(DISTINCT immigrations_table.airport_code) as flight_count
FROM immigrations_table
JOIN airports_table
ON airports_table.airport_code = immigrations_table.airport_code
GROUP BY airports_table.state_code
ORDER BY flight_count DESC
LIMIT 10;
```

```
SELECT airports_table.city_name, count(immigrations_table.airport_code) as flight_count, time_table.month
FROM immigrations_table
JOIN airports_table
ON airports_table.airport_code = immigrations_table.airport_code
JOIN time_table
ON immigrations_table.arrival_date = time_table.date
GROUP BY time_table.month, airports_table.city_name
ORDER BY flight_count DESC
LIMIT 30;
```

## Data Model
![Alt text](data_model_udend_capstone.png?raw=true)

## Data Dictionary
[Data Dictionary](DataDictionary.pdf?raw=true)

## Project Setup - One time actions
In order to run this data pipeline, you will first need to ensure you are set up with the appropriate data sets. Then you will need to create a bucket in S3 that will be used as your data lake. The last AWS piece is a redshift cluster. Once these are created, fill in the dl.cfg with the AWS key credentials, details on the redshift cluster, details on the S3 bucket, and the IAM role that will be used when performing the copy from S3 to redshift. Then we will do the table creations only once, as outlined below in steps to run.

## Steps to Run
To run this project, run ‘python create_tables.py’. Then run ‘python etl.py' to perform all necessary steps for the pipeline including data quality checks. If you need to re-run it, you will need to manually delete the artifcats in S3, this ensures it does not accidentally overwrite the parquet tables in S3 and ensure purposeful data lake changes.
### Repeatable Pipeline Steps
1. Load staging tables with minimally filtered data to S3 as parquet
2. Perform some transformations and load target tables to S3
3. Copy staging and target tables into Redshift


## What’s Next and Other Possible Scenarios
The next step would be to add a table to redshift containing the mapping of the values from the I94. This would give more insight to some of the columns from the immigrations_table such as residence and citizenship. Also, one of the constraints of this project is that theer are target tables in S3 and in redshift, which means we may want to add more to the data quality checks to ensure the table remain in unison.

#### 100x data
If next data load required 100x more data I would increase the redshift cluster to a ds2.8xlarge to accomodate the large size and tune the spark jobs to be on a more distributed system by adding more nodes to the compute processes. Since we are already using Spark, we have very efficient memory allocation already in place to handle a large distributed system for that much data.

#### Daily pipeline
If the pipeline had to run at 7am every day, I would definitely utilize a scheduler such as airflow. I would also need to add refreshing the s3 bucket folders for the data lake. Within airflow we would complete all of the steps outlined above in the pipeline. I would also definitely want to add a check to ensure the data in S3 is in sync with what we have in redshift, unless we have a need to further aggregate data in redshift which is possibility with this project.

#### Access to 100+ people
I would probably use Mesos as a cluster manager if many different team members are going to be working on the database. Also, we would need to again increase the redshift size as mentioned above in order to accomodate more users. Additionally I would enable the concurrency scaling feature of redshift which would enable concurrent users and queries to the data warehouse without losing performance on query speed.
