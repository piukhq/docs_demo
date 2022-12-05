The Bink DWH is built within a dedicated Snowflake environment, and is populated with an ELT process. Airbyte is used to extract the data from Bink source systems and DBT (Data build Tool) is used to transform the raw data into usable fact and dimension tables. The whole process is orchestrated by Prefect.

## Snowflake

### Structure

The dedicated snowflake instance has three separate databases: raw, dev, and bink.

-   BINK is the production database, where tables are staged, transformed, and presented in their respective schemas. There are also analytics schemas here for logging the results of dbt runs and tests.
    -   BINK & BINK_SECURE are the output schemas, in which all analysis should be performed.
-   DEV is a development database, with the same basic structure as prod.
-   RAW is the endpoint for data extraction, containing schemas for the different data sets.

### Roles

The role structure for the warehouse is as follows, in decreasing levels of access ability:

-   AccountAdmin: Top level role. Manages all aspects of account.
-   Manager: Data manger. Can access RAW and BINK.
-   SeniorAnalyst: Access to BINK, including BINK_SECURE, but not to RAW.
-   Analyst: Access to BINK, but not to BINK_SECURE.
-   JuniorAnalyst: Access to BINK, but not to BINK_SECURE (currently identical to Analyst role).

In addition there is a ConnectRole which Airbyte uses for extraction and a TransformRole used by DBT.

To create a new user, login with account admin privileges, and run the following code in a Snowflake worksheet, changing out the necessary variables.

```sql
begin;
    -- role variable
   set junior_role = 'JUNIOR_ANALYST' ;
   set mid_role = 'ANALYST';
   set senior_role = 'SENIOR_ANALYST';
   set manager_role = 'MANAGER';
   
   set connect_role = 'CONNECT_ROLE';
   set transform_role = 'TRANSFORM_ROLE';
    
    -- new user variables
   set login_ =  '<user email>';
   set first_ = '<user first name>';
   set last_ = '<user last name>';
   set email_ = '<user email>';
   set comment_ = '<user role description>';
      
   -- warehouse variables
   set warehouse_name = 'ANALYTICS';   
   set database_name = 'BINK';
        
   -- create user
   create user if not exists identifier($user_)
   password = ''
   login_name = $login_
   FIRST_NAME = $first_
   LAST_NAME = $last_
   EMAIL = $email_
   comment = $comment_
   default_role = $mid_role -- change this as required
   default_warehouse = $warehouse_name;

   grant role identifier($mid_role) to user identifier($user_); -- change this as required
   
 commit;
```

The new user will also need to be added to the Snowflake users group in Azure AD.

## DBT

DBT is used to migrate the data from raw through staging tables, an optional transform layer, and finally output to finished tables. See DBT docs [here](https://docs.getdbt.com/docs/introduction "https://docs.getdbt.com/docs/introduction") for more information.

The code for DBT is contained within the 'Bink' directory in the repo, and essentially comprises descriptions, in SQL, of the different data models. DBT also contains tests, which are found both in the DBT yaml files and as individual tests within the test directory.

The build also makes use of the Elementary package to provide tests and to log the output of DBT. This information is outputted into the Bink_monitoring schema.

DBT is run on the prefect server. For a standard ETL run, DBT is triggered in the following order:

1.  Test source (Tests run on the raw data)
2.  Run (Build models)
3.  Output testing (Sanity check on data output)
4.  Business tests (More involved tests which warn, as opposed to fail the run)

Prefect, see below, is set up to only build the models if source testing passes.

### Docs

DBT can generate visualisation of data models. To do this, run `dbt docs generate; dbt docs serve` within the Bink directory. This will build and serve up a locally hosted set of data documentation.

## Airbyte

Airbyte is used to extract the data out of the source systems and load into the raw layer of the datawarehouse. Because there are three sources, there are three connections set up to load data from hermes, harmonia, and rabbitmq events. These connections are set to manual trigger as Prefect is responsible for starting the daily load.

Prefect is deployed on a VM within the Bink azure subscription. The Prefect VM IP must be whitelisted.

## Prefect

Prefect is the orchestration tool used to trigger Airbtye extractions and DBT runs. At a set time, a Prefect flow is set off, and works through a DAG, which is defined by python code contained in the repo.

Prefect can be used with Prefect Server or it’s managed service form, Prefect Cloud. We recommend setting up Prefect Server on a cloud hosted VM. Please see the attached documentation on how to do that.