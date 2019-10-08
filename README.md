# UDEND_Capstone

The data being used for this project is I94 Immigration Data in the SAS format and the Airport data in CSV. The goal of the project is to see what the biggest hubs are for immigration travel. Below are the staging tables for this data.

Steps for pipeline:
Load filtered data as parquet files into S3
Create staging tables in redshift
Load staging tables from S3 to redshift
Create target tables in redshift
Transform & load data from staging tables to target tables in S3
Copy target tables to Redshift

## Goal
The goal of this project is to use immigration and airport data to help a immigration focused non profit succeed. Initially, and on a very tight deadline, they want to see what airports are most popular amongst immigrants to the USA. This data will fuel decisions around how to best assist immigrants in their travels and work within the states to better staff their offices around the major immigration hubs. The project uses spark to make a data lake in S3, as well as load table into redshift. Redshift will create a data warehouse for the analytics team to get what they need regarding immigration airport hubs quickly. The data lake in S3 along with the staging tables is for future analysis, since there was also a requirement from the non profit to keep any valuable data pertaining to the immigration events. For now we will keep the data in the immigrations_data relatively raw, so that we can get what we need regarding the airport hub very quickly. Next steps for the data are discussed below.

For this scenario, we wouldn’t really need data updated too often, probably only annually since it will impact something that can’t be changed all too quickly. However, the other unknown use cases for the data lake may require the data to be updated more often. 

If I were going to increase the data 100x, I would use a cluster.

If the pipeline had to run at 7am every day, I would definitely utilize a scheduler such as airflow. I would also need to add refreshing the s3 bucket folders for the data lake.

If the database needed to be accessed by over 100 people I think I made a good choice in using redshift, as it lends itself to this requirement.

## Data Model
![Alt text](data_model_udend_capstone.png?raw=true)

## Data Dictionary
![Alt text](DataDictionary.pdf?raw=true)

## Project Setup
In order to run this data pipeline, you will first need to ensure you are set up with the appropriate data sets. Then you will need to create a bucket in S3 that will be used as your data lake. The last AWS piece is a redshift cluster. Once these are created, fill in the dl.cfg with the AWS key credentials, details on the redshift cluster, details on the S3 bucket, and the IAM role that will be used when performing the copy from S3 to redshift.


## Project Run
To run this project, run ‘python create_tables.py’. Then run ‘python etl.py' to perform all necessary steps for the pipeline including data quality checks. If you need to re-run it, you will need to manually delete the artifcats in S3, this ensures it does not accidentally overwrite the parquet tables in S3 and ensure purposeful data lake changes.


## What’s Next?
The next step would be to add a table to redshift containing the mapping of the values from the I94. This would give more insight to some of the columns from the immigrations_table such as residence and citizenship. 
