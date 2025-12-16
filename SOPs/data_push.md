# Data Push

## Purpose
The purpose of this SOP is to provide guidance on how to move data from ETL through testing and into production.

## Prequirements
- NIH-NCI account
- NIH laptop w/ admin rights & NIH-VPN access
- Access to the CloudOne Secrets Manager 
    - Process described in SOP: [obtaining_cloudone_database_credentials.md](obtaining_cloudone_database_credentials.md)
- Installation of **psql** on NIH laptop
    - > **_NOTE:_** All procedures listed in this document can instead be run within a SQL management application such as DBeaver

## Procedures

1. Download the ETL SQL dump file to your local machine
1. Connect to the NIH-VPN via the Cisco Secure Client application
1. Connect to the postgres instance and create a temporary database to upload the new data:
    - In a Terminal window:
        1. Log into the postgres instance via:
            ``` bash
            psql -h host -U username -d postgres
            ```
            - > **_NOTE:_** Use the **host**, **username**, and **password** values from the Secrets Manager
            - > **_NOTE:_** At this step, we are connecting to the **“postgres”** database not the **dbname** database
        1. Create the temporary database via:
            ``` sql
            CREATE DATABASE temp_db;
            ```
        1. Exit the postgres session via:
            ``` sql
            \q
            ```
1. Upload the SQL dump to the temporary database:
    - In a Terminal window:
        1. Navigate to the directory where the ETL's SQL dump file is located
        1. Upload the dump file to the temporary database via:
            ``` bash
            psql -h host -U username -d temp_db -W < DUMPFILE.sql
            ```
             - > **_NOTE:_** Use the values from Secrets Manager again and change **DUMPFILE.sql** to the actual name of the file
        1. Wait for the upload to complete
            - > **_NOTE:_** This should take between 5-20 minutes
            - > **_NOTE:_** Ignore any ERRORs about roles not existing
1. Rename the old database to something else and change the temporary one to the **dbname** as seen in the Secrets Manager
    - In a Terminal window:
        - > **_NOTE:_** Change dbname in the following commands to the one from Secrets Manager
        1. Log into the postgres instance via:
            ``` sql
            psql -h host -U username -d postgres
            ```
        1. Prevent additional connections to the databases while you are renaming via:
            ``` sql
            ALTER DATABASE dbname with ALLOW_CONNECTIONS false;
            ```
        1. And again with the temp_db via:
            ``` sql
            ALTER DATABASE temp_db with ALLOW_CONNECTIONS false;
            ```
        1. Terminate all active connections to the two databases via:
            ``` sql
            SELECT pg_terminate_backend(pid)
            FROM pg_stat_activity
            WHERE (datname = 'dbname' or datname = 'temp_db') AND pid <> pg_backend_pid();
            ```
        1. Rename the old database via:
            ``` sql
            ALTER DATABASE dbname RENAME TO old_db;
            ```
        1. Rename the temp database to dbname via:
            ``` sql
            ALTER DATABASE temp_db RENAME TO dbname;
            ```
        1. Reallow connections to the dbname database via:
            ``` sql
            ALTER DATABASE dbname with ALLOW_CONNECTIONS true;
            ```
        1. Do not exist the postgres session yet
1. Perform smoke testing 
    - > **_NOTE:_** Follow guidance in SOP: [cda_testing.md](cda_testing.md)
1. Once verified, you may remove the “old_db” via:
    ``` sql 
    DROP DATABASE old_db;
    ```
    - > **_NOTE:_** It may be beneficial to wait on this step for a week or so to ensure that there are no major problems
1. Exit the postgres session via:
    ```
    \q
    ```
