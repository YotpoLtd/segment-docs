---
title: Snowflake Reverse ETL Setup
beta: true
redirect_from:
  - '/reverse-etl/snowflake-setup/'
---

Set up Snowflake as your Reverse ETL source. 

At a high level, when you set up Snowflake for Reverse ETL, the configured user/role needs read permissions for any resources (databases, schemas, tables) the query needs to access. Segment keeps track of changes to your query results with a managed schema <br>(`__SEGMENT_REVERSE_ETL`), which requires the configured user to allow write permissions for that schema.

> success ""
> Segment now supports key-pair authentication for Snowflake Reverse ETL sources. Key-pair authentication is available for Business Tier users only.

## Set up guide
Follow the instructions below to set up the Segment Snowflake connector. Segment recommends you use the `ACCOUNTADMIN` role to execute all the commands below, and that you create a user that authenticates with an encrypted key pair.

1. Log in to your Snowflake account.
2. Navigate to *Worksheets*.
3. Enter and run the code below to create a database.
   Segment uses the database specified in your connection settings to create a schema called `__segment_reverse_etl` to avoid collision with your data. The schema is used for tracking changes to your model query results between syncs.
   An existing database can be reused, if desired. Segment recommends you to use the same database across all your models attached to this source to keep all the state tracking tables in 1 place.

   ```sql
   -- not required if another database is being reused
   CREATE DATABASE segment_reverse_etl;
   ```
4. Enter and run the code below to create a virtual warehouse.
   Segment Reverse ETL needs to execute queries on your Snowflake account, which requires a Virtual Warehouse to handle the compute. You can also reuse an existing warehouse.

   ```sql
   -- not required if reusing another warehouse
   CREATE WAREHOUSE segment_reverse_etl
    WITH WAREHOUSE_SIZE = 'XSMALL'
      WAREHOUSE_TYPE = 'STANDARD'
      AUTO_SUSPEND = 600 -- 5 minutes
      AUTO_RESUME = TRUE;
   ```
5. Enter and run the code below to create specific roles for Reverse ETL.
   All Snowflake access is specified through roles, which are then assigned to the user you’ll create later.

   ```sql
   -- create role
   CREATE ROLE segment_reverse_etl;

   -- warehouse access
   GRANT USAGE ON WAREHOUSE segment_reverse_etl TO ROLE segment_reverse_etl;

   -- database access
   GRANT USAGE ON DATABASE segment_reverse_etl TO ROLE segment_reverse_etl;
   GRANT CREATE SCHEMA ON DATABASE segment_reverse_etl TO ROLE segment_reverse_etl;
   ```
6. Enter and run one of the following code snippets below to create the user Segment uses to run queries. For added security, Segment recommends creating a user that authenticates using a key pair.

   To create a user that authenticates with a key pair, [create a key pair](https://docs.snowflake.com/en/user-guide/key-pair-auth#configuring-key-pair-authentication){:target="_blank”} and then execute the following SQL commands: 
   ``` sql
   -- create user (key-pair authentication)
   CREATE USER segment_reverse_etl_user
   DEFAULT_ROLE = segment_reverse_etl
   RSA_PUBLIC_KEY = 'enter your public key';

   -- role access
   GRANT ROLE segment_reverse_etl TO USER segment_reverse_etl_user;
   ```

   To create a user that authenticates with a password, execute the following SQL commands:
   ```sql
   -- create user (password authentication)
   CREATE USER segment_reverse_etl_user
    MUST_CHANGE_PASSWORD = FALSE
    DEFAULT_ROLE = segment_reverse_etl
    PASSWORD = 'my_strong_password'; -- Do not use this password

   -- role access
   GRANT ROLE segment_reverse_etl TO USER segment_reverse_etl_user;
   ```
7. Follow the steps listed in the [Add a Source](/docs/connections/reverse-etl#step-1-add-a-source) section to finish adding Snowflake as a source.
