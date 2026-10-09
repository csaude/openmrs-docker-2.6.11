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

Clone the [`sis-rme-data-migration`](https://github.com/MISAU-DIS/sis-rme-data-migration) repository to a directory on the host where the OpenMRS Docker Compose deployment runs.

If the repository has already been cloned, use the existing local copy.

git clone https://github.com/MISAU-DIS/sis-rme-data-migration.git

### 2. Configure the ETL directory

Create the `.env` file from `.env.sample`, if it does not already exist:

cp .env.sample .env

Open `.env` and locate the commented `sync_etl_dir` entry. Uncomment it and set it to the `etl` directory inside the repository cloned in Step 1.

# Directory containing the SIS-RME data-migration ETL files
sync_etl_dir=<path-to-cloned-repository>/etl

Replace `<path-to-cloned-repository>` with the actual path to the cloned repository. The path may be absolute or relative to the directory containing the Docker Compose file.

For example, if the repository was cloned to `/opt/sis-rme-data-migration`, configure:

sync_etl_dir=/opt/sis-rme-data-migration/etl

Verify that the configured directory contains the required ETL files, including the module and synchronization configuration:

etl/
├── app/
│   └── etl-1.0.omod
└── conf/
    └── sync/
        └── sis_rme_sesp_sync.json

### 3. Enable the Docker Compose volume mounts

In `docker-compose.yml`, locate the `refapp-tomcat` service and its `volumes` section.

The following volume mounts are already present but commented out. Uncomment both entries:


# ETL configuration, templates and scripts
- ${sync_etl_dir}:/opt/openmrs/etl

# ETL OpenMRS module (OMOD)
- ${sync_etl_dir}/app/etl-1.0.omod:/usr/local/tomcat/.OpenMRS/modules/etl-1.0.omod

These mounts make the ETL configuration files available inside the OpenMRS container and provide the ETL module to OpenMRS.

### 4. Validate the configuration and start OpenMRS

From the directory containing the Docker Compose file, validate the configuration:

docker compose config

Confirm that both ETL volume mounts resolve to the expected host paths and that the OMOD file exists.

Start the services:

docker compose up -d

On the first startup, OpenMRS must load the ETL module so that its Global Properties become available for configuration.

The `epts.etl.enabled` Global Property defaults to `false`. Leave this default unchanged during the initial setup so that synchronization does not start before the required configuration has been verified.

### 5. Configure the ETL Global Properties

After OpenMRS has loaded the ETL module, access the OpenMRS administration interface and navigate to **Administration → Settings → EPTS**.

In the EPTS module settings, configure the Global Properties required for the synchronization to work in the target environment.

#### 5.1. Database connection properties

The ETL module connects to two databases:

* **Source:** `openmrs_event_receiver_mgt`, which stores synchronization messages produced by NiFi.
* **Destination:** `openmrs`, the OpenMRS database into which the data is synchronized.

Under normal deployment conditions, the source database should already exist and its message table should be populated by the NiFi pipeline.

For information on configuring the NiFi pipeline, refer to the **SIS-RME/NiFi integration guide**: [Insert link to the NiFi integration guide].

Update and verify the following Global Properties in the EPTS module settings:

| Global Property             | Purpose                                                     |
| --------------------------- | ----------------------------------------------------------- |
| `epts.etl.srcConnectionURI` | JDBC connection URI for the source database                 |
| `epts.etl.srcUserName`      | Username used to connect to the source database             |
| `epts.etl.srcUserPassword`  | Password used to connect to the source database             |
| `epts.etl.dstConnectionURI` | JDBC connection URI for the OpenMRS destination database    |
| `epts.etl.dstUserName`      | Username used to connect to the destination database        |
| `epts.etl.dstUserPassword`  | Password used to connect to the destination database        |
| `epts.etl.src_db`           | Source database name; normally `openmrs_event_receiver_mgt` |
| `epts.etl.dst_db`           | Destination database name; normally `openmrs`               |

The connection URIs must use a hostname or IP address and port reachable from the Tomcat container. Use the appropriate address and port for the deployment environment.

Example connection URIs:

epts.etl.srcConnectionURI=jdbc:mysql://<database-host>:<database-port>/openmrs_event_receiver_mgt?autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true
epts.etl.dstConnectionURI=jdbc:mysql://<database-host>:<database-port>/openmrs?autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true

Replace `<database-host>` and `<database-port>` with the correct database address and port. If the databases run in Docker, use the appropriate hostname and port for the network configuration.

Set the database usernames and passwords to valid credentials with the required permissions. Do not retain default credentials in a production environment or commit real passwords to the repository.

#### 5.2. Other synchronization properties

Review and configure the other Global Properties required by the deployment. Depending on the synchronization templates, these may include:

| Global Property                         | Purpose                                                   |
| --------------------------------------- | --------------------------------------------------------- |
| `epts.etl.sync_destination_location_id` | Destination location ID                                   |
| `epts.etl.hiv.destination.location.id`  | Destination location ID for HIV synchronization           |
| `epts.etl.smi.destination.location.id`  | Destination location ID for SMI synchronization           |
| `epts.etl.src.location.list`            | List of source location IDs or codes                      |
| `epts.etl.tarv.program.id`              | Destination TARV program ID                               |
| `epts.etl.prep.program.id`              | Destination PrEP program ID                               |
| `epts.etl.ccr.program.id`               | Destination CCR program ID                                |
| `epts.etl.dst_tarv_service_id`          | Destination TARV service ID                               |
| `epts.etl.dst_prep_service_id`          | Destination PrEP service ID                               |
| `epts.etl.dst_ccr_service_id`           | Destination CCR service ID                                |
| `epts.etl.defaultUserId`                | Default OpenMRS user ID, if required by the configuration |

Use the correct IDs for the target OpenMRS installation. Do not copy values from another environment without verifying them.

Before proceeding, confirm that:

* The ETL configuration and startup files are available inside the container.
* The source database exists and receives messages from NiFi.
* The source and destination connection URIs, credentials, and database names are correct.
* The required location, program, service, and user IDs are configured for the target environment.
* `epts.etl.enabled` remains `false` until the configuration has been verified.

### 6. Activate synchronization

Once all required Global Properties have been configured and verified, return to the EPTS module settings and change:

epts.etl.enabled=true

Save the change, then restart the Tomcat container:

docker restart refapp-tomcat

Allow OpenMRS and the ETL module to start. With `epts.etl.enabled=true` and the required configuration in place, synchronization should start automatically.

### 7. Verify synchronization in the OpenMRS interface

In the OpenMRS module list, locate the synchronization dashboard and navigate to:

**Sincronização → Painel de Sincronização SIS-RME → SESP**

Use this dashboard to verify that synchronization events are being processed and to inspect pending or failed records.

A running Docker container alone does not confirm that synchronization is working correctly. Confirm that events are being processed through the dashboard and investigate any pending or failed records before considering the activation complete.
