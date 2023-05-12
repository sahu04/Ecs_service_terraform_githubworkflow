# DSI-Automation

This repository using for the process automate  transformation of table data from a PostgreSQL database and subsequently export it as CSV files. These CSV files can then be easily uploaded to an S3 bucket.The process involves establishing a connection to the PostgreSQL database, executing queries to retrieve the desired table data, and formatting the data into CSV format.

## Repository Contains

- [Jenkinsfile](#Jenkinsfile)
- [Dockerfile](#Dockerfile)
- [export_data.py](#export_data.py)
- [tables.yaml](#tables.yaml)
- [taskdef_template.json](#taskdef_template.json)


## Prerequisites

The following artifacts must be setup prior to running the code:
* AWS CLI. 
* RDS PostgreSQL Database. 
* Python script (connect to RDS postgresSQL Database table convert to CSV format)
* yaml script (specify column name and table details)
* S3 bucket that will contain RDS PostgresSQL data under the `data-dump/` Folder in CSV Format.
* Secret manager (entry with name "DSI-postgress-DB" to load env variables)

##  tables.yaml script
this script contains the tables and columns deatsils from RDS PostgreSQL database table The structure of the YAML file is designed to define a list of tables, where each table has a name and a list of associated columns.

```
tables:
  - name: table_name
    columns:
      - column1
      - column2
```

## Python script

Python file containing the code. This file serves as the main script responsible for connecting to the PostgreSQL database, executing queries, exporting data to CSV files, and uploading them to the specified S3 bucket.

## Installation

Install Python Modules to run script

```
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
* `RDS_DBNAME`: RDS PostgreSQL dbname
* `SOURCE_BUCKET`: Source Bucket name 
* `DESTINATION_BUCKET`: Destination Bucket name
* `DESTINATION_PREFIX`: Destination Prefix


    

