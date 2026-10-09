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

## **## SIS-RME to OpenMRS Synchronization Activation**

## SIS-RME to OpenMRS Synchronization Activation

This section describes how to activate SIS-RME to OpenMRS synchronization using the ETL module.

### 1. Clone the data-migration repository

Clone the `sis-rme-data-migration` repository to a directory on the host where the OpenMRS Docker Compose deployment runs.

If the repository has already been cloned, use the existing local copy.

git clone <sis-rme-data-migration-repository-url>

### 2. Configure the ETL directory

Edit the `.env` file of this Docker Compose project and configure `sync_etl_dir` to point to the `etl` directory inside the previously cloned `sis-rme-data-migration` repository.

# Directory containing the SIS-RME data-migration ETL files
sync_etl_dir=<path-to-cloned-repository>/etl

Replace `<path-to-cloned-repository>` with the actual path to the repository cloned in Step 1. The path may be absolute or relative to the directory containing the Docker Compose file.

For example, if the repository was cloned to `/opt/sis-rme-data-migration`, configure:

sync_etl_dir=/opt/sis-rme-data-migration/etl

The configured ETL directory must contain the ETL configuration files, templates, scripts, and OMOD module, including:

etl/
├── app/
│   └── etl-1.0.omod
└── conf/
    └── sync/
        └── sis_rme_sesp_sync.json

### 3. Configure Docker Compose volume mounts

Ensure that the `refapp-tomcat` service contains the following volume mounts:

# ETL configuration, templates and scripts
- ${sync_etl_dir}:/opt/openmrs/etl

# ETL OpenMRS module
- ${sync_etl_dir}/app/etl-1.0.omod:/usr/local/tomcat/.OpenMRS/modules/etl-1.0.omod

These mounts provide the ETL files and module to the OpenMRS container.

### 4. Verify the configuration

From the directory containing the Docker Compose file, verify that the configured ETL directory and OMOD file exist.

Validate the resolved Compose configuration:

docker compose config

Ensure that both ETL mounts resolve to the expected host paths before proceeding.

### 5. Verify ETL Global Properties

In OpenMRS, verify that the ETL Global Properties point to the correct configuration paths:

| Global Property           | Expected value                                      |
| ------------------------- | --------------------------------------------------- |
| `epts.etl.enabled`        | `true`                                              |
| `epts.etl.mode`           | `db_synchronization`                                |
| `epts.etl.conf.dir`       | `/opt/openmrs/etl/conf`                             |
| `epts.etl.etl_root_dir`   | `/opt/openmrs/etl`                                  |
| `epts.etl.startup.file`   | `/opt/openmrs/etl/conf/sync/sis_rme_sesp_sync.json` |
| `epts.etl.sync_fragments` | `sync/sesp-sync-fragments/*.json`                   |

Also verify the source and destination database connection properties, credentials, destination location ID, and other required synchronization properties.

### 6. Start and verify synchronization

Apply the Docker Compose configuration:

docker compose up -d

If the ETL module is newly installed or has been updated, ensure that OpenMRS loads it successfully. A controlled Tomcat restart may be required.

Monitor the Tomcat logs:

docker logs -f --tail 100 refapp-tomcat

Confirm that the ETL module loads, the startup configuration and synchronization fragments are found, database connections succeed, and synchronization events are processed without unexpected errors.

A running container does not, by itself, confirm that synchronization is working correctly. Verify the ETL logs and event processing status.
