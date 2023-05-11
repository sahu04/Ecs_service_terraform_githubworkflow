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

### Environment Variables

The following environment variable needs to be set in prior to execution. On AWS, the service will look for secret manager entry with name "DSI-postgress-DB" to load variables.

* `LDF_VSAMB_RDS_HOST`: Aurora PostgreSQL host
* `LDF_VSAMB_RDS_USERNAME`: Aurora PostgreSQL username
* `LDF_VSAMB_RDS_PASSWORD`: Aurora PostgreSQL password
* `LDF_VSAMB_S3_BULK_LOAD_DATA`: name of loader S3 bucket (should be the same bucket as vault-search-bulk-loader)
* `LDF_VSAMB_ENV`: running environment (develop, staging, preprod, prod). This MUST BE SET on each environment accordingly because the data loaded differ from one environment to another.

The SQL database that must be created prior: `vault_app_manager`

## Run

### Application executable

To run program, simply go to cmd/ and run the following command:

* `go build`

This will build an executable that you can then run.


## Endpoints:

The loader service exposes these three endpoints:

* `GET` `/health` health check to return service state
* `POST` `/v1/migration` to perform database migration
* `POST` `/v1/loader` to perform load operation
* `GET` `/v1/status` to return status of running operation. A `ready` response indicate that no jobs are currently running. A response value matching
one of the mode values [`migration`, `loader`] indicates that the corresponding operation is currently running.

## Load operation sequence:
To load, follow below sequence of endpoints.
1) `POST` `/v1/migration` First we need to call this endpoint to perform db migration
2) `POST` `/v1/loader` In second step we need to call this endpoint

## Check status of migration or Loader:
* `GET` `/v1/status` Once migration or load process is started, call this endpoint to check status of running jobs. If response contain string "ready" that means the last job is complete. You can call this endpoint at any time to check current status. Please note that you can only run a single operation at a time (for example, if migration is in progress and you attempt to call loader endpoint, the call will fail with 409 error).

In case of failures in `/v1/loader`, if you need to start over you will have to delete the database and re-run the load operation sequence. `Migration` and `Loader` operations are idempotent; they can be run any number of times.

### Centralized Approvers List
An SSM Parameter store has been created in the `shared-ou` account. Update the SSM Parameter store to add or remove approvers list.

Below is the SSM Parameter store names of DS team and AAIC team

DS-DevOps Team: `/Jenkins/Vault-Teams/DS-DevOps/ApproversList`

AAIC-DevOps Team: `/Jenkins/Vault-Team/AAIC-DevOps/ApproversList`

