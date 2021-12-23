+++
title = "Compare DAG API structure of workflow tools"
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


## Azure Data Factory

## Airflow

https://airflow.apache.org/docs/apache-airflow/2.0.2/usage-cli.html#exporting-dag-structure-as-an-image

### View DAG tree

```bash
docker-compose run airflow-worker airflow dags show example_bash_operator
```

## Databricks