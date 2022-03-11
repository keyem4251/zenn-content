---
title: "AirflowでローカルのPythonファイルのimport、syntaxテスト"
emoji: "⛑"
type: "tech"
topics: ["airflow", "test"]
published: true
---

# はじめに
Aiflowをクラウド環境（Composerなど）で動かす際にクラウド環境の他のサービスと連携する部分のテストが行えずとりあえず、Dagファイルをアップロードしてimport、syntaxのエラーが出ることがあると思います。
そんなときに公式のドキュメントに載っている方法ではありますが、一番シンプルにPythonのvenvを使用してDagファイルのimport、syntaxのエラーが起きていないかの確認を行うための方法を記載します。  
公式のドキュメントではDag Loader Testの部分にあたります。(公式ドキュメント)[https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html#dag-loader-test]

# 環境
macOS Big Sur 11.6.2  
チップ Apple M1

# 実行
まずはPythonをインストールします。  
ここではクラウド環境を用いる前提で扱うバージョンによりPythonのバージョンも変わると思うので、pyenvを使用します。  
pyenvのインストールについては他の記事を参考にしてください。
```
$ pyenv install 3.8.12
$ pyenv local 3.8.12
```

requirements.txtファイルを作成します。  
ここではComposerを使用する前提でGCPのサービスと連携する場合のためにgoogleのproviderもインストールします。（他に必要なモジュールがある場合は適宜インストールしてください）
```
apache-airflow==2.1.4
apache-airflow-providers-google==6.4.0
```
venv環境を作成します。（make、シェルファイルにしておくといいかもしれません）
```
$ python3 -m venv venv
$ source venv/bin/activate
$ pip install --upgrade pip
$ pip install -r requirements.txt
$ deactivate
```
dagsフォルダを作成します。
```
$ mkdir dags
```
dagsフォルダ下にairflowで実行するためのファイルを作成します。airflowの公式に載っているものの一部のimportをエラーするようにしています [(参考: Airflow公式サンプルコード)](https://github.com/apache/airflow/blob/main/airflow/providers/google/cloud/example_dags/example_bigquery_sensors.py)
```
"""
Example Airflow DAG for Google BigQuery Sensors.
"""
import os
from datetime import datetime

from airflow import models
from airflow.providers.google.cloud.operators.bigquery import (
    BigQueryCreateEmptyDatasetOperator,
    BigQueryCreateEmptyTableOperator,
    BigQueryDeleteDatasetOperator,
    BigQueryInsertJobOperator,
)
from airflow.providers.google.cloud.sensors.bigquery import (
    BigQueryTableExistenceSensor,
)

PROJECT_ID = os.environ.get("GCP_PROJECT_ID", "example-project")
DATASET_NAME = os.environ.get("GCP_BIGQUERY_DATASET_NAME", "test_sensors_dataset")

TABLE_NAME = "partitioned_table"
INSERT_DATE = datetime.now().strftime("%Y-%m-%d")

PARTITION_NAME = "{{ ds_nodash }}"

INSERT_ROWS_QUERY = f"INSERT {DATASET_NAME}.{TABLE_NAME} VALUES " "(42, '{{ ds }}')"

SCHEMA = [
    {"name": "value", "type": "INTEGER", "mode": "REQUIRED"},
    {"name": "ds", "type": "DATE", "mode": "NULLABLE"},
]

dag_id = "example_bigquery_sensors"

with models.DAG(
    dag_id,
    schedule_interval='@once',  # Override to match your needs
    start_date=datetime(2021, 1, 1),
    catchup=False,
    tags=["example"],
    user_defined_macros={"DATASET": DATASET_NAME, "TABLE": TABLE_NAME},
    default_args={"project_id": PROJECT_ID},
) as dag_with_locations:
    create_dataset = BigQueryCreateEmptyDatasetOperator(
        task_id="create-dataset", dataset_id=DATASET_NAME, project_id=PROJECT_ID
    )

    create_table = ABigQueryCreateEmptyTableOperator(
        task_id="create_table",
        dataset_id=DATASET_NAME,
        table_id=TABLE_NAME,
        schema_fields=SCHEMA,
        time_partitioning={
            "type": "DAY",
            "field": "ds",
        },
    )
    # [START howto_sensor_bigquery_table]
    check_table_exists = BigQueryTableExistenceSensor(
        task_id="check_table_exists", project_id=PROJECT_ID, dataset_id=DATASET_NAME, table_id=TABLE_NAME
    )
    # [END howto_sensor_bigquery_table]

    execute_insert_query = BigQueryInsertJobOperator(
        task_id="execute_insert_query",
        configuration={
            "query": {
                "query": INSERT_ROWS_QUERY,
                "useLegacySql": False,
            }
        },
    )

    delete_dataset = BigQueryDeleteDatasetOperator(
        task_id="delete_dataset", dataset_id=DATASET_NAME, delete_contents=True
    )

    create_dataset >> create_table
    create_table >> check_table_exists
    create_table >> execute_insert_query
```

シンタックスエラーなどがないかテストします。  
今回はエラーが出るようにimportしているクラスの名前を変えているので、エラーが出ています。
```
$ source venv/bin/activate
$ python3 dags/
$ deactivate

Traceback (most recent call last):
  File "dags/sample.py", line 48, in <module>
    create_table = ABigQueryCreateEmptyTableOperator(
NameError: name 'ABigQueryCreateEmptyTableOperator' is not defined
```
このようにvenv環境を作るだけでも手軽にPythonファイルの構文エラーを確認できます。  
あくまで今回は手軽にということを重点においていますので、お使いの環境に適したテストの仕方をしていただければと思います。（docker-composeやhelmなどなど）
