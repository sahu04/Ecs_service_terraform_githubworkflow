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

# Klarity Database management

This repository store migration scripts for Klarity PostgreSQL database.

Migrations are executed through https://github.com/pressly/goose, itself launched within a container.

The complete schema can be found under https://github.com/demandscience/klarity-services/blob/DEVELOP/scripts/datamodel/schema.sql .

For more information about this, please check https://github.com/demandscience/klarity-services#database-model

## Repository structure

1. New SQL migration files need to be added into `db-goose-migration/db` folder
2. each file should be added in incremental numbers 
    e.g. `0001_migration.sql`, `0002_migration.sql`, etc.
```
├── db-goose-migration
│   ├── db
│   │   ├── 001_initial_schema.sql
│   │   ├── 002_LNG-751_administrator_audit_log.sql
│   │   └── ***.sql
│   ├── Dockerfile
│   ├── Dockerfile.down
│   ├── goose_script_down_level.sh
│   └── goose_script.sh
├── devops
│   ├── do-release.sh
│   └── driver.config
├── do-release.sh
├── Jenkinsfile
├── README.md
└── taskdef_template_db.json

```

## What is within a Python script

A Python script is a single file whose name is prefixed by an incrementing sequence number (named the schema version).

Each script is composed of 2 sections:
- an `Up` section, responsible to migrate an existing database version the version N-1 to version N (version taken from the filename prefix);
- and a `Down` section, which is responsible of doing the revert operation.

The `Up` section must be included within the following block:

```
-- +goose Up
-- +goose StatementBegin
SELECT 'up LNG-XXX';

-- SQL statements goes here

-- +goose StatementEnd
```

Similarly, the `Down` section happens within the following block:

```
-- +goose Down
-- +goose StatementBegin
SELECT 'down LNG-XXX';

-- SQL statements goes here

-- +goose StatementEnd
```

Migration is executed within a transaction. This imply that operations like `ANALYZE` cannot be executed by migration scripts. And if an extensive data migration is done, locks will be retained during the complete migration.


## Testing migrations locally

Two Docker compose files exists to test the migration scripts locally:
- `docker-compose-up.yaml` which applies all migration scripts from an empty database until the latest version of the schema;
- `docker-compose-down.yaml` which does the operation in the reverse order, migrating a database from the latest version to the initial script version `001`.

### Test the up path

Simply run the following command to create a new empty PostgreSQL instance and apply all migration scripts

```
docker compose -f docker-compose-up.yaml up --build
```

The compose file waits 15 seconds for the database to be ready before running the migration. This can be controlled by the environment variable `WAIT_TIME`.

## Test the down path

Testing the down path requires an existing container DB that is already migrated (through the up path). So, after testing the up path with the section above, use the following command:

```
docker compose -f docker-compose-down.yaml up --build
```


## Build deployment container

Building the container locally requires access to DemandScience Container Registry (ECR) which sits within the "SHARED-OU" organizational unit. If you don't have access to this OU, or if you lack the "Access-to-ECR" permission, please reach out to DevOps to request access.

First install the latest AWS CLI version (Amazon ECR functionality is available in the AWS CLI starting with version 1.9.15). You can check your AWS CLI version with the `aws --version` command. 

For information about installing the AWS CLI or upgrading it to the latest version, visit this link:

[https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)

Export the credentials from awsapps.com web portal. This will set the following environment variables for you:

- `AWS_ACCESS_KEY_ID`

- `AWS_SECRET_ACCESS_KEY`

- `AWS_SESSION_TOKEN`

You also need to set the following extra environment variables:

- `export AWS_DEFAULT_REGION=us-east-2`

- `export AWS_ACCOUNT_ID=141758851966`

After you have installed and configured the AWS CLI, authenticate the Docker CLI. That way, the docker command can push and pull images with Amazon ECR. The AWS CLI provides a get-login-password command to simplify the authentication process.

`aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com`

#### In order to build a deployable container use the following command:
Use this command for Goose Up:

`docker build -t klarity-db -f ./db-goose-migration/Dockerfile --build-arg AWS_REGION="$AWS_DEFAULT_REGION" --build-arg ACCOUNT_ID="$AWS_ACCOUNT_ID" ./db-goose-migration`

Use this command for Goose Down/Rollback:

`docker build -t klarity-db -f ./db-goose-migration/Dockerfile.down --build-arg AWS_REGION="$AWS_DEFAULT_REGION" --build-arg ACCOUNT_ID="$AWS_ACCOUNT_ID" ./db-goose-migration`

## Release management

Releases of Klarity follows the principles and guidelines documented under https://www.notion.so/gardenshed/Releases-management-b45f71034f3147f8bf5f81d5d17a9e03 .

All changes merged into the `DEVELOP` branch are built automatically. Execution of the migration pipeline requires one approval, which is prompted within the Slack channel [`#klarity-db-ci-cd`](https://demandscience.slack.com/archives/C042KLEN801).

To promote a version of Klarity to another environment, please use the script `./devops/do-release.sh`.

This script optionally rely on [GitHub CLI](https://cli.github.com/). While not stricly required, having it installed and ready for use makes the release process even more simple and automated.

To prepare a new release, you need the following information:
- the source branch
- the destination branch
- if and only if the source branch is `DEVELOP`, the next version number

### Procedure

1. Locally checkout the source branch
1. Run the script `./devops/do-release.sh`
1. The script will prompt you for the destination branch: enter either, `STAGING`, `PRE-PROD` or `PROD`
1. If the source branch is `DEVELOP`, the script will prompt you for the next version number
1. The script will create a new delivery branch, merge the changes, push the branch to GitHub and create the delivery pull request (automatically if you have GitHub CLI installed, otherwise you'll need to do it manually)
   - :warning: if the delivery branch already exists, the script will prompt you to delete it
   - :warning: if the merge fails, the script stops and you have to manually complete the procedure
1. Once the branch is created, don't forget to request for reviews (one approval is required for `STAGING`, 2 otherwise)
1. If the source branch is `DEVELOP`, the script will create a version upgrade branch, modify the `devops/driver.config` file with the newer version, push the branch to GitHub and create a version ugprade pull request
1. Again, this pull request must be manually assigned reviewers. It is also a good practice to merge it as soon as possible.
