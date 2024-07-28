# DSI-Automation

This repository is used  for establishing connection into PostgreSQL database. After that, the queries are executed to retrieve the desired table data. The retrieved data is formatted into CSV format and the resulting CSV files are uploaded to S3 bucket.

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
this script contains the tables and columns deatails from RDS PostgreSQL database table The structure of the YAML file is designed to define a list of tables, where each table has a name and a list of associated columns.

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
* `SOURCE_BUCKET`: Source Bucket name (Prod)
* `DESTINATION_BUCKET`: Destination Bucket name(Non-Prod)
* `DESTINATION_PREFIX`: Destination Prefix

## Jenkinsfile
 when we export data into S3 Bucket `destination prefix` variable should be updated in the Jenkinsfile.
```
{enviroment
 DESTINATION_PREFIX = "data-dump/MMYYYY"
}
   ``` 


