# Docker Compose Configuration for OpenMRS 2.6.11 Deployment
-
Project can be used to either run an 2.6.11 instance or upgrade an 2.3.x instance (it uses the OpenMRS platform 2.6.11 war file). In order to launch/run, one is required to provide an sql dump of desired OpenMRS instance. When first run, the openmrs database will be created and then populated with the data from the provided sql dump. The subsequent run however, will skip this step.

## Steps to run

1. Edit and customize .env.sample file (using your editor of choice nano, vim, ...) save.
2. Copy the updated .env.sample to .env.
       **cp .env.sample .env** 

3. Place the SQL dump of the database which has to be named _openmrs.sql_ in _**/database/scripts**_ directory.
4. While in the project's directory run the command `docker compose up -d`.

**Note:** Don't forget to provide the required SQL dump file.



# Docker Compose Configuration for OpenMRS 2.6.11 Deployment and dbsync
1. Place the SQL dump of the database which has to be named _openmrs.sql_ in _**/database/scripts**_ directory.
2. Make sure you follow the instruction in this guide to set dbsync properties
   
   [EIP README](./dbsync/README.md)
    
3. Start the project 

```
    docker compose -f docker compose dbdsync up -d

````




Follow the container logs using

```
docker logs --follow openmrs-eip-sender
```

## **## SIS-RME to OpenMRS Synchronization Installation**

This section describes how to configure SIS-RME to OpenMRS synchronization using the ETL module.

### 1. Configure the ETL directory

Configure the `sync_etl_dir` variable in the `.env` file to point to the `etl` directory of the `sis-rme-data-migration` repository.

```dotenv
sync_etl_dir=etl_mig_directory
```

Replace `etl_mig_directory` with the actual absolute or relative path to the ETL directory in your environment.

The directory must contain the ETL configuration files, templates, scripts, and the module file at:


etl/
├── app/
│   └── etl-1.0.omod
└── conf/
    └── sync/
        └── sis_rme_sesp_sync.json


### 2. Configure Docker Compose

Ensure that the `refapp-tomcat` service contains the following volume mounts:


- ${sync_etl_dir}:/opt/openmrs/etl
- ${sync_etl_dir}/app/etl-1.0.omod:/usr/local/tomcat/.OpenMRS/modules/etl-1.0.omod

These mounts make the ETL configuration available to the container and provide the ETL OMOD module to OpenMRS.

### 3. Validate the configuration

Before starting the services, verify that the ETL directory and OMOD file exist at the configured location.

Validate the resolved Docker Compose configuration:

docker compose config

Ensure that both ETL volume mounts resolve to the expected host paths.

### 4. Configure ETL Global Properties

Verify that the ETL Global Properties in OpenMRS point to the correct configuration paths:

| Property                  | Value                                               |
| ------------------------- | --------------------------------------------------- |
| `epts.etl.enabled`        | `true`                                              |
| `epts.etl.mode`           | `db_synchronization`                                |
| `epts.etl.conf.dir`       | `/opt/openmrs/etl/conf`                             |
| `epts.etl.etl_root_dir`   | `/opt/openmrs/etl`                                  |
| `epts.etl.startup.file`   | `/opt/openmrs/etl/conf/sync/sis_rme_sesp_sync.json` |
| `epts.etl.sync_fragments` | `sync/sesp-sync-fragments/*.json`                   |

Also verify the source and destination database connection properties, credentials, destination location ID, and any required program and service IDs.

### 5. Start and verify synchronization

Start or update the services:

docker compose up -d

Follow the Tomcat logs:

docker logs -f --tail 100 refapp-tomcat

Confirm that the ETL module loads successfully, the startup configuration and synchronization fragments are found, the database connections succeed, and synchronization events are processed without unexpected errors.

A running container alone does not confirm that synchronization is working correctly; always verify the ETL logs and processing status.

