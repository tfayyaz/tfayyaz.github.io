+++
title = "Analysing the Airflow Metadata Database"
date = "2021-12-20T14:43:41Z"
author = "Tahir Fayyaz"
authorTwitter = "tfayyaz" #do not include @
cover = "Test Cover"
tags = ["apache-airflow", "sql"]
keywords = ["Apache Airflow", "SQLlite"]
description = "blah blah blah"
showFullContent = false
readingTime = false
draft = true
+++

Data quality monitoring and observabilty is a growing space [[1](https://www.castordoc.com/blog/data-monitoring-and-observability), [2](https://notion.castordoc.com/catalog-of-data-quality), [3](https://www.metaplane.dev/state-of-data-quality-monitoring-2021)] in data and analytics engineering and following this trend lead me to explore some of the ways you can extract metadata and logs from popular data tools. The first tool, Apache Airflow, is one I have used for a few years and so I knew the Airflow metadata database contains a lot of useful information about DAGs, operators, job runs and users. To showcase what is possible and also to get a better understanding of what can be done with access to the metadata I set-up a local Airflow instance, enabled some of the example DAGs included and carried out some exploratory data analysis which is all shared below.

An important note is that what you install Airflow locally using pip it uses a SQLite database which is perfect for tests like this but for production your metadata database will most likely using MySQL or Postgres, as is recomended by Airflow. The good news is that the schema of the databases will be the same and so a lot of these SQL queries should eaisly port over.

For these tests I am going to use Airflow 2.0.2 as that is the version used by 

## Set-up Airflow locally using Docker

```bash
mkdir airflow-2-0-2
cd airflow-2-0-2
```

Follow all the instructions here to get Airflow and Docker up and running. The Airflow community have really made it simple and fast now - https://airflow.apache.org/docs/apache-airflow/2.0.2/start/docker.html

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.0.2/docker-compose.yaml'
```

In `docker-composer.yaml` change `AIRFLOW__CORE__LOAD_EXAMPLES: 'true'` to `AIRFLOW__CORE__LOAD_EXAMPLES: 'false'` to avoid loading all the example DAGs.

-  image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:2.0.1}
+ image: ${AIRFLOW_IMAGE_NAME:-apache/airflow:2.0.1-python3.8}

```bash
docker-compose up airflow-init
```

After this you should see:

```
airflow-init_1       | Upgrades done
airflow-init_1       | Admin user airflow created
airflow-init_1       | 2.0.2
start_airflow-init_1 exited with code 0
```

### Enable DAGs

Enable the following DAGs:

- 1
- 2. DAG
- 3 dag

### List DAGs


```bash
docker-compose run airflow-worker airflow dags list
```

### View DAG tree

```bash
docker-compose run airflow-worker airflow dags show example_bash_operator
```

### Run back-fill for DAGs


```bash
docker-compose run airflow-worker airflow dags backfill \
    --start-date '2021-11-01' \
    --end-date '2021-12-01' \
    example_bash_operator
```

```bash
docker-compose run airflow-worker airflow dags backfill \
    --start-date '2021-12-01' \
    --end-date '2021-12-07' \
    example_subdag_operator
```

### Access Postgres CLI

Check Postgres is running and get conatiner name

```bash
docker ps
```

This can be done using:

```bash
docker exec -ti airflow-2-0-2_postgres_1 psql -U airflow
docker exec -ti <CONTAINER_NAME> psql -U airflow
docker-compose run airflow-worker airflow db shell
```

Since Airflow 2.2 it seems it is even easier to get Airflow running locally.

```bash
airflow setup
mkdir airflow
cd airflow
python3 -m venv env
source env/bin/activate

# AIRFLOW_HOME=$(pwd)
export AIRFLOW_HOME=~/airflow_dev/airflow
export AIRFLOW_HOME=~/test_airflow/airflow

# Install Airflow using the constraints file
AIRFLOW_VERSION=2.2.2

PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
# For example: 3.6
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
# For example: https://raw.githubusercontent.com/apache/airflow/constraints-2.2.2/constraints-3.6.txt
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

airflow standalone
```

Once you run `airflow standalone` you will get a URL to access the webserver and log-in details as follows:

```
standalone | 
standalone | Airflow is ready
standalone | Login with username: admin  password: xn10gGz4FaqztzK
standalone | Airflow Standalone is for development purposes only. Do not use this in production!
standalone | 
```

The constraints in the constraint.txt file are only installed if a package requires them, and are ignored otherwise.

```bash
airflow setup
mkdir airflow
cd airflow
pipenv --python 3.8
pipenv shell
export AIRFLOW_HOME=$(pwd)
pipenv install apache-airflow==2.1.0
pipenv install apache-airflow-providers-databricks
mkdir dags
airflow db init
airflow users create --username admin --firstname <firstname> --lastname <lastname> --role Admin --email your@email.com
```

## Copy some of the example DAGs 

```
export AIRFLOW_HOME=~/airflow_dev/airflow
export AIRFLOW_HOME=~/test_airflow/airflow
cd ~/test_airflow/airflow
mkdir dags
```

## Access the Postgres Database

## Change Postgres settings to show full output

\pset pager off 

## Access the SQLlite Metadata database

Open a new shell terminal tab and start the virtual env

```bash
source env/bin/activate
```

Locally it's as simple as running the following cmd


```bash
airflow db shell
```

set headers to on

```sql
.mode column
.headers on
```

## Explore and analyse Airflow Metadata

### View all tables in database

```sql
.tables
```

output:

```bash
ab_permission                  import_error                 
ab_permission_view             job                          
ab_permission_view_role        log                          
ab_register_user               rendered_task_instance_fields
ab_role                        sensor_instance              
ab_user                        serialized_dag               
ab_user_role                   sla_miss                     
ab_view_menu                   slot_pool                    
alembic_version                task_fail                    
connection                     task_instance                
dag                            task_reschedule              
dag_code                       trigger                      
dag_pickle                     variable                     
dag_run                        xcom                         
dag_tag                         
```

### View schema of a table

This is useful to also see how the tables can be joined with other tables based the `FOREIGN KEYS`

```sql
.schema task_instance
```

Output:

```sql
CREATE TABLE IF NOT EXISTS "task_instance" (
	task_id VARCHAR(250) NOT NULL, 
	dag_id VARCHAR(250) NOT NULL, 
	start_date DATETIME, 
	end_date DATETIME, 
	duration FLOAT, 
	state VARCHAR(20), 
	try_number INTEGER, 
	hostname VARCHAR(1000), 
	unixname VARCHAR(1000), 
	job_id INTEGER, 
	pool VARCHAR(256) NOT NULL, 
	queue VARCHAR(256), 
	priority_weight INTEGER, 
	operator VARCHAR(1000), 
	queued_dttm DATETIME, 
	pid INTEGER, 
	max_tries INTEGER DEFAULT '-1', 
	executor_config BLOB, 
	pool_slots INTEGER NOT NULL, 
	queued_by_job_id INTEGER, 
	external_executor_id VARCHAR(250), 
	trigger_id INTEGER, 
	trigger_timeout DATETIME, 
	next_method VARCHAR(1000), 
	next_kwargs JSON, 
	run_id VARCHAR(250) NOT NULL, 
	CONSTRAINT task_instance_pkey PRIMARY KEY (dag_id, task_id, run_id), 
	CONSTRAINT task_instance_trigger_id_fkey FOREIGN KEY(trigger_id) REFERENCES "trigger" (id) ON DELETE CASCADE, 
	CONSTRAINT task_instance_dag_run_fkey FOREIGN KEY(dag_id, run_id) REFERENCES dag_run (dag_id, run_id) ON DELETE CASCADE
);
CREATE INDEX ti_dag_state ON task_instance (dag_id, state);
CREATE INDEX ti_state ON task_instance (state);
CREATE INDEX ti_trigger_id ON task_instance (trigger_id);
CREATE INDEX ti_pool ON task_instance (pool, state, priority_weight);
CREATE INDEX ti_job_id ON task_instance (job_id);
CREATE INDEX ti_dag_run ON task_instance (dag_id, run_id);
CREATE INDEX ti_state_lkp ON task_instance (dag_id, task_id, run_id, state);
```

## Queries

- Most Used Operators - DONE
- Longest Running DAGs - DONE
- DAG run time by slot - DONE
- No of DAGs using xcom
- No of DAGs using sensors
- Cross DAG dependencies

### Show all dag IDs and operator name

```sql
select dag_id, operator from task_instance group by dag_id, operator;
```

```bash
sqlite> select dag_id, operator from task_instance group by dag_id, operator;

```

Result:

airflow_profiler|SqliteOperator
----------------|---------------
example_bash_operator|BashOperator
example_bash_operator|DummyOperator
example_external_task_marker_child|DummyOperator
example_external_task_marker_child|ExternalTaskSensor
example_external_task_marker_parent|ExternalTaskMarker
example_subdag_operator|DummyOperator
example_subdag_operator|SubDagOperator
example_subdag_operator.section-1|DummyOperator
example_task_group|BashOperator
example_task_group|DummyOperator
example_trigger_controller_dag|TriggerDagRunOperator
example_trigger_target_dag|BashOperator
example_trigger_target_dag|PythonOperator
example_xcom|PythonOperator
latest_only_with_trigger|DummyOperator
latest_only_with_trigger|LatestOnlyOperator

### Show most popular operators

```
select operator, count(distinct dag_id) from task_instance group by operator;
```

Result:

operator|count
--------|------
BashOperator|4
BranchDateTimeOperator|1
BranchDayOfWeekOperator|1
BranchPythonOperator|3
DummyOperator|12
DummySkipOperator|1
PythonOperator|2
ShortCircuitOperator|1
SubDagOperator|1
TriggerDagRunOperator|1
_PythonDecoratedOperator|1

### Longest running DAGs last 30 days

```sql
with dag_succesful_runs as (
  select
    date(start_date) as run_date,
    dag_id,
    cast (
      (
        julianday(end_date) - julianday(start_date)
      ) * 24 * 60 * 60 * 1000 * 1000 as integer
    ) as dag_run_microseconds
  from
    dag_run
  where
    start_date > datetime('now', '-30 day')
    and state = 'success'
)
select
  dag_id,
  count(dag_id) as no_dag_runs,
  sum(dag_run_microseconds) / 1000 / 1000 as sum_run_time,
  avg(dag_run_microseconds) / 1000 / 1000 as avg_run_time
from
  dag_succesful_runs
group by
  dag_id
order by
  avg_run_time desc;
```

### Show total and avg time for each dag

```sql
select * from dag_run where dag_id = 'schedule_external_task_marker_parent_v2';
```

```sql
select dag_id, (end_date - start_date) as task_time from dag_run where dag_id = 'example_bash_operator';
```

```sql
select dag_id, 
cast ((
    julianday(end_date) - julianday(start_date)
) * 24 * 60 * 60 as integer) as dag_run_seconds
from dag_run where dag_id = 'schedule_external_task_marker_parent_v2';
```

### Calculate task run time in milliseconds

```sql
select dag_id, task_id, end_date, julianday(end_date), start_date,
cast ((
    julianday(end_date) - julianday(start_date)
) * 24 * 60 * 60 * 1000 as integer) as task_run_milliseconds,
duration as task_duration
from task_instance where dag_id = 'schedule_external_task_marker_parent_v2';
```

### Show total DAG run time per day

```sql
select
  dag_run_date,
  sum(dag_run_seconds) as total_run_seconds
from
  (
    select
      date(start_date) as dag_run_date,
      cast (
        (julianday(end_date) - julianday(start_date)) * 24 * 60 * 60 as integer
      ) as dag_run_seconds
    from
      dag_run
  );
```

### Show total DAG time per slot pool

The pool used is not avaiable in the `dag_run` table so you need to use the `task_instance` table which also has the task duration calculated.

Postgres:

```sql

select
  pool,
  dag_id || '_' || job_id as dag_job_id,
  min(start_date) as min_start_date,
  max(end_date) as max_end_date
from
  task_instance
group by pool, dag_job_id
order by dag_job_id;


select
  pool,
  concat_ws('_', dag_id, run_id) as dag_run_id,
  min(start_date) as min_start_date,
  max(end_date) as max_end_date
from
  task_instance
group by dag_run_id
order by dag_run_id;
```

```
     pool     |          dag_job_id           |        min_start_date         | 
        max_end_date          
--------------+-------------------------------+-------------------------------+-
------------------------------
 default_pool | example_bash_operator_10      | 2021-12-23 12:06:27.895559+00 | 
2021-12-23 12:06:28.070771+00
 default_pool | example_bash_operator_11      | 2021-12-23 12:06:29.299855+00 | 
2021-12-23 12:06:29.502291+00
 default_pool | example_bash_operator_2       | 2021-12-23 12:06:25.655889+00 | 
2021-12-23 12:06:27.223989+00
 default_pool | example_bash_operator_3       | 2021-12-23 12:06:25.685703+00 | 
2021-12-23 12:06:27.278723+00
 default_pool | example_bash_operator_4       | 2021-12-23 12:06:25.780904+00 | 
2021-12-23 12:06:27.426265+00
 default_pool | example_bash_operator_5       | 2021-12-23 12:06:25.837548+00 | 
2021-12-23 12:06:26.413039+00
 default_pool | example_bash_operator_6       | 2021-12-23 12:06:26.7895+00   | 
2021-12-23 12:06:27.117059+00
 default_pool | example_bash_operator_7       | 2021-12-23 12:06:26.806193+00 | 
2021-12-23 12:06:28.134296+00
 default_pool | example_bash_operator_8       | 2021-12-23 12:06:26.825538+00 | 
2021-12-23 12:06:28.159168+00
 default_pool | example_bash_operator_9       | 2021-12-23 12:06:26.85047+00  |

```

```sql
with dag_runs as (
select
  pool,
  dag_id || "_" || run_id as dag_run_id,
  min(start_date) as min_start_date,
  max(end_date) as max_end_date
from
  task_instance
group by pool, dag_run_id
order by dag_run_id
),
dag_run_times as (
select 
  pool,
  min_start_date,
  max_end_date,
  cast ((
    julianday(max_end_date) - julianday(min_start_date)
) * 24 * 60 * 60 as integer) as dag_run_seconds,
  cast ((
    julianday(max_end_date) - julianday(min_start_date)
) * 24 * 60 * 60 * 1000 as integer) as dag_run_milli_seconds,
  cast ((
    julianday(max_end_date) - julianday(min_start_date)
) * 24 * 60 * 60 * 1000 * 1000 as integer) as dag_run_micro_seconds
from dag_runs
)

select 
  date(min_start_date) as run_date,
  pool,
  sum(dag_run_micro_seconds) / 1000 / 1000 as dag_run_seconds
from dag_run_times
group by run_date, pool
;
```


### Analysing sensors

Run the DAG example_external_task_marker_child manually and wait for 30 seconds then run the DAG example_external_task_marker_parent as this 