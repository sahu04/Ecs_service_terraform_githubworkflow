# DSI-Automation

This repository using for RDS PostgreSQL database Table  data upload to S3 bucket in CSV Formate.


## Repository Contains

- [Jenkinsfile](#Jenkinsfile)
- [Dockerfile](#Dockerfile)
- [export_data.py](#export_data.py)
- [table.yaml](#table.yaml)
- [taskdef_template.json](#taskdef_template.json)


## Prerequisites

The following artifacts must be setup prior to running the code:

* RDS PostgreSQL Database. 
* Python script (connect to RDS postgresSQL Database table conver to CSV formate)
* yaml script (specify column name and table details)
* S3 bucket that will contain RDS PostgresSQL data under the `data-dump/` Folder in CSV Formate.
* Secret manager (entry with name "DSI-postgress-DB" to load env variables)

##  table.yaml script
this script contains the tables and columns deatsils from RDS PostgreSQL database table :

```
tables:
  - name: table_name
    columns:
      - column1
      - column2
```

## What is within a Python script

Python script that exports data from a PostgreSQL database hosted on Amazon RDS (Relational Database Service) to CSV files and uploads them to an S3 bucket. 

## Installation

Install Python Modules to run script

```bash
 pip install psycopg2
 pip install boto3
 pip install pyyaml

```
Retrive values from Enviroment variable:
The following environment variable needs to be set in prior to execution. On AWS, the service will look for secret manager entry with name "DSI-postgres-DB" to load variables.

* `RDS_HOST`: RDS PostgreSQL host
* `RDS_USERNAME`: RDS PostgreSQL username
* `RDS_PASSWORD`: RDS PostgreSQL password
* `RDS_PORT`: RDS PostgreSQL port


connect to s3 bucket with AWS credentials execute query:

* `S3_Bucket`: S3 bucket name

Load the tables and columns from the YAML file

```bash
  with open('tables.yaml', 'r') as f:
    tables = yaml.safe_load(f)['tables']
```
    
Upload the data from PostgrSQL database in formate CSV file to S3_Bucket

```bash
  logging.info(f'Uploading {name}.csv to S3...')
    s3_client.upload_file(f'{name}.csv', BUCKET_NAME, f'Bucket_folder/{name}.csv')
```

