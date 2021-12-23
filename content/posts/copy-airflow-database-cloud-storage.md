+++
title = "Copy Airflow Metadata Database to Cloud Storage"
date = "2021-12-23T14:43:41Z"
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

The airflow metadata database can get quite large which is why you have tool like https://github.com/teamclairvoyant/airflow-maintenance-dags/tree/master/db-cleanup. If the database is going to be cleaned but you need a back-up you can query here are some ways to copy your database to cloud storage ready to be queried.

Back-up Database first if you do not want to read from production DB:

pgsql dump

```bash
docker exec -t airflow-2-0-2_postgres_1 pg_dumpall -c -U airflow > dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql
```

source: https://stackoverflow.com/questions/24718706/backup-restore-a-dockerized-postgresql-database


## Docker export to CSV

```bash
docker exec -t airflow-2-0-2_postgres_1 psql -U airflow -c "COPY task_instance TO STDOUT WITH CSV HEADER" > dump_task.csv
```

source: https://stackoverflow.com/questions/69704933/export-query-result-as-csv-file-from-docker-postgresql-container-to-local-machin
