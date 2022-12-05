## Run Commands

### Run

DBT is run using the command `dbt run`. This command runs all of the data models in order of where they fit in the DAG.

[https://docs.getdbt.com/reference/commands/run](https://docs.getdbt.com/reference/commands/run)

## Model Types

Models are the individual .SQL files stored in the dbt project. They compile when run into objects within snowflake. Either tables, view or temporary tables.

### Table

The table type creates a table within snowflake. The table is rebuilt from the ground up on each run.

### View

The view type makes a view within snowflake.

### incremental

The incremental type creates a table that is merged after the initial run that creates the table. A time field like `updated_at` is required to use this kind of table. It by default uses snowflakes merge statement to append new data

Code snippet of an incremental table. The code within `{% if is_incremental() %}` is only run after the initial run.

To refresh incremental models form the start the command dbt run --full-refresh must be run

```sql
select * from all_events

-- if the table already exists and `--full-refresh` is
-- not set, then only add new records. otherwise, select
-- all records.
{% if is_incremental() %}
   where collector_tstamp > (
     select coalesce(max(max_tstamp), '0001-01-01') from {{ this }}
   )
{% endif %}
```

### ephemeral

Ephemeral makes a temporary table that only exists for the duration of the session. This table is now used in this project as it makes it more difficult to debug.

## Logs

### Pass/Success

Success means that model has run successfully and moves on to the next model downstream.

### Error

Error means there was an issue when running the code. Models with error messages will have the errors printed at the bottom

### Skip

Models are skipped if an upstream model has an error message making it impossible to run the downstream model

```
09:07:36  Running with dbt=1.0.1
09:07:39  Found 63 models, 183 tests, 0 snapshots, 0 analyses, 573 macros, 2 operations, 0 seed files, 23 sources, 0 exposures, 0 metrics
09:07:39  
09:07:44  Concurrency: 1 threads (target='dev')
09:07:44  
09:07:44  1 of 63 START view model BINK_STAGING.stg_harmonia__loyalty_scheme.............. [RUN]
09:07:45  1 of 63 OK created view model BINK_STAGING.stg_harmonia__loyalty_scheme......... [SUCCESS 1 in 0.67s]

09:07:47  5 of 63 START incremental model BINK_STAGING.stg_hermes__EVENTS................. [RUN]
09:07:48  5 of 63 ERROR creating incremental model BINK_STAGING.stg_hermes__EVENTS........ [ERROR in 1.10s]

09:08:24  23 of 63 SKIP relation BINK_SECURE.fact_loyalty_card_add_auth_secure............ [SKIP]
09:08:24  24 of 63 SKIP relation BINK_SECURE.fact_loyalty_card_auth_secure................ [SKIP]
09:08:24  25 of 63 SKIP relation BINK_SECURE.fact_loyalty_card_join_secure................ [SKIP]
09:08:24  26 of 63 SKIP relation BINK_SECURE.fact_loyalty_card_register_secure............ [SKIP]
09:08:24  27 of 63 SKIP relation BINK_SECURE.fact_loyalty_card_removed_secure............. [SKIP]
09:08:24  28 of 63 SKIP relation BINK.fact_user........................................... [SKIP]

09:09:19
09:09:19  
09:09:19  Finished running 17 view models, 19 incremental models, 27 table models, 2 hooks in 99.93s.
09:09:19  
09:09:19  Completed with 1 error and 0 warnings:
09:09:19
09:09:19  Database Error in model stg_hermes__EVENTS (models\staging\hermes_events\stg_hermes__EVENTS.sql)
09:09:19    000904 (42000): SQL compilation error: error line 38 at position 3
09:09:19    invalid identifier '_AIRBYTE_UNIQUE_KEY'
09:09:19
09:09:19  Done. PASS=42 WARN=0 ERROR=1 SKIP=20 TOTAL=63
```
