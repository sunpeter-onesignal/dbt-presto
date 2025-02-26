## dbt-presto

### Documentation
For more information on using Presto with dbt, consult the dbt documentation:
- [Presto profile](https://docs.getdbt.com/docs/profile-presto)

### Installation
This plugin can be installed via pip:
```
$ pip install dbt-presto
```

### Configuring your profile

A dbt profile can be configured to run against Presto using the following configuration:

| Option  | Description                                        | Required?               | Example                  |
|---------|----------------------------------------------------|-------------------------|--------------------------|
| method  | The Presto authentication method to use | Optional (default is `none`)  | `none` or `kerberos` |
| user  | Username for authentication | Required  | `drew` |
| password  | Password for authentication | Optional (required if `method` is `ldap` or `kerberos`)  | `none` or `abc123` |
| database  | Specify the database to build models into | Required  | `analytics` |
| schema  | Specify the schema to build models into. Note: it is not recommended to use upper or mixed case schema names | Required | `dbt_drew` |
| host    | The hostname to connect to | Required | `127.0.0.1`  |
| port    | The port to connect to the host on | Required | `8080` |
| threads    | How many threads dbt should use | Optional (default is `1`) | `8` |



**Example profiles.yml entry:**
```
my-presto-db:
  target: dev
  outputs:
    dev:
      type: presto
      user: drew
      host: 127.0.0.1
      port: 8080
      database: analytics
      schema: dbt_drew
      threads: 8
```

### Usage Notes

#### Supported Functionality
Due to the nature of Presto, not all core dbt functionality is supported.
The following features of dbt are not implemented on Presto:
- Archival
- Incremental models

Also, note that upper or mixed case schema names will cause catalog queries to fail. Please only use lower case schema names with this adapter.

If you are interested in helping to add support for this functionality in dbt on Presto, please [open an issue](https://github.com/fishtown-analytics/dbt-presto/issues/new)!

#### Required configuration
dbt fundamentally works by dropping and creating tables and views in databases.
As such, the following Presto configs must be set for dbt to work properly on Presto:

```
hive.metastore-cache-ttl=0s
hive.metastore-refresh-interval = 5s
hive.allow-drop-table=true
hive.allow-rename-table=true
```

#### Use table properties to configure connector specifics

Trino/Presto connectors use table properties to configure connector specifics.

Check the Presto/Trino connector documentation for more information.

```
{{
  config(
    materialized='table',
    properties={
      "format": "'PARQUET'",
      "partitioning": "ARRAY['bucket(id, 2)']",
    }
  )
}}
```

### Reporting bugs and contributing code

-   Want to report a bug or request a feature? Let us know on [Slack](http://slack.getdbt.com/), or open [an issue](https://github.com/fishtown-analytics/dbt-presto/issues/new).

### Running tests
Build dbt container locally:

```
./docker/dbt/build.sh
```

Run a Presto server locally:

```
./docker/init.bash
```

If you see errors while about "inconsistent state" while bringing up presto,
you may need to drop and re-create the `public` schema in the hive metastore:
```
# Example error

Initialization script hive-schema-2.3.0.postgres.sql
Error: ERROR: relation "BUCKETING_COLS" already exists (state=42P07,code=0)
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
Underlying cause: java.io.IOException : Schema script failed, errorcode 2
Use --verbose for detailed stacktrace.
*** schemaTool failed ***
```

**Solution:** Drop (or rename) the public schema to allow the init script to recreate the metastore from scratch. **Only run this against a test Presto deployment. Do not run this in production!**
```sql
-- run this against the hive metastore (port forwarded to 10005 by default)
-- DO NOT RUN THIS IN PRODUCTION!

drop schema public cascade;
create schema public;
```

You probably should be slightly less reckless than this.

Run tests against Presto:

```
./docker/run_tests.bash
```

Run the locally-built docker image (from docker/dbt/build.sh):
```
export DBT_PROJECT_DIR=$HOME/... # wherever the dbt project you want to run is
docker run -it --mount "type=bind,source=$HOME/.dbt/,target=/home/dbt_user/.dbt" --mount="type=bind,source=$DBT_PROJECT_DIR,target=/usr/app" --network dbt-net dbt-presto /bin/bash
```

## Code of Conduct

Everyone interacting in the dbt project's codebases, issue trackers, chat rooms, and mailing lists is expected to follow the [PyPA Code of Conduct](https://www.pypa.io/en/latest/code-of-conduct/).
