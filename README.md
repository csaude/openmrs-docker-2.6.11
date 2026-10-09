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

## SIS-RME to OpenMRS Synchronization Activation

This section describes how to activate SIS-RME to OpenMRS synchronization using the ETL module.

### 1. Clone the data-migration repository

Clone the [sis-rme-data-migration repository](https://github.com/MISAU-DIS/sis-rme-data-migration) to a directory on the host where the OpenMRS Docker Compose deployment runs.

If the repository has already been cloned, use the existing local copy.

git clone https://github.com/MISAU-DIS/sis-rme-data-migration.git

### 2. Configure the ETL directory

First, create the .env file by copying .env.sample in the Docker Compose project directory:

cp .env.sample .env

Edit the newly created .env file and configure sync_etl_dir to point to the etl directory inside the previously cloned sis-rme-data-migration repository.


# Directory containing the SIS-RME data-migration ETL files
sync_etl_dir=<path-to-cloned-repository>/etl

Replace `<path-to-cloned-repository>` with the actual path to the cloned repository. The path may be absolute or relative to the directory containing the Docker Compose file.

For example, if the repository was cloned to `/opt/sis-rme-data-migration`, configure:

sync_etl_dir=/opt/sis-rme-data-migration/etl

Ensure that the configured ETL directory contains the required configuration files, templates, scripts, and OMOD module, including:

etl/
├── app/
│   └── etl-1.0.omod
└── conf/
    └── sync/
        └── sis_rme_sesp_sync.json

### 3. Configure Docker Compose volume mounts

In the `docker-compose.yml` file, locate the `refapp-tomcat` service and its `volumes` section.

The ETL volume mount lines are already present in the configuration but may be commented out. Uncomment both lines to enable the ETL configuration and OMOD mounts:

# ETL - sis-rme-data-migration
- ${sync_etl_dir}:/opt/openmrs/etl

# ETL - ETL module (OMOD)
- ${sync_etl_dir}/app/etl-1.0.omod:/usr/local/tomcat/.OpenMRS/modules/etl-1.0.omod

These entries must be placed under the `volumes` section of the `refapp-tomcat` service, alongside the existing volume mounts.

### 4. Verify the configuration

From the directory containing the Docker Compose file, verify that the ETL directory and OMOD file exist at the configured locations.

Validate the resolved Docker Compose configuration:

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

#### Monitor synchronization through the OpenMRS interface

In OpenMRS, navigate to the synchronization monitoring interface module:

**Sincronização
Painel de Sincronização SIS-RME → SESP** also **SIS-RME → SESP Synchronization Dashboard**

Use the **SIS-RME → SESP Synchronization Dashboard** to monitor the synchronization process and verify that events are being processed.

Confirm that the synchronization is running as expected and check for pending events, processing errors, or failed records, as applicable.

A running container does not, by itself, confirm that synchronization is working correctly. Always verify the synchronization status through the monitoring dashboard.

