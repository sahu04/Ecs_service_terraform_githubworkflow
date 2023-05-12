# DSI-Automation

This repository using for RDS PostgreSQL database Table  data upload to S3 bucket in CSV Formate.


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
* Python script (connect to RDS postgresSQL Database table convert to CSV formate)
* yaml script (specify column name and table details)
* S3 bucket that will contain RDS PostgresSQL data under the `data-dump/` Folder in CSV Formate.
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
* `RDS_DBNAME`: RDS PostgreSQL dbname
* `SOURCE_BUCKET`: Source Bucket name 
* `DESTINATION_BUCKET`: Destination Bucket name
* `DESTINATION_PREFIX`: Destination Prefix

connect to s3 bucket with AWS credentials execute query:
```
  BUCKET_NAME=SOURCE_BUCKET
  s3_client = boto3.client('s3')

```

Load the tables and columns from the`tables.yaml` scripts.

```
  with open('tables.yaml', 'r') as f:
    tables = yaml.safe_load(f)['tables']
```
    
 Loop through the tables and columns, Data export them to CSV format
 ```
for table in tables:
    name = table['name']
    columns = ','.join(table['columns'])
     logging.info(f'Exporting table {name}...')

```
 database tables data  rows & Column upload  to a CSV file
```
 with open(f'{name}.csv', 'w') as f:
        # Write the header row
        header = [col.strip() for col in columns.split(',')]
        f.write(','.join(header) + '\n')

 rows = cur.fetchall()
        for row in rows:
            row = [str(col) if ',' not in str(col) else f'"{col}"' for col in row]
            f.write(', '.join(row) + '\n') 
  
 
```

Upload the data from PostgrSQL database in formate CSV file to S3_Bucket

```
logging.info(f'Uploading {name}.csv to S3...')
    s3_client.upload_file(f'{name}.csv', BUCKET_NAME, f'{DESTINATION_PREFIX}/{name}.csv')
```
    


Sync database tables data into s3 bucket with another S3 Bucket:

`Prefix`: Bucket_folder name

```
    logging.info(f'Syncing S3 bucket {BUCKET_NAME} with another bucket...')
    for obj in s3_client.list_objects(Bucket=BUCKET_NAME, Prefix='Bucket_folder')['Contents']:
        obj_key = obj['Key']
        dest_key = obj_key.replace('Bucket_folder', '')
        copy_source = {'Bucket': BUCKET_NAME, 'Key': obj_key}
        s3_client.copy_object(Bucket=DESTINATION_BUCKET, CopySource=copy_source, Key=f'{DESTINATION_PREFIX}/{dest_key}')
```
    

