---
title: "Cloud Composerのローカル開発 composer-local-dev お試し"
emoji: ""
type: "tech"
topics: ["airflow", "GCP", "Composer"]
published: false
---
# はじめに
Cloud Composerをローカル環境で動作確認を行うことができる[composer-local-dev](https://cloud.google.com/composer/docs/composer-2/run-local-airflow-environments)というものがリリースされたので試してみます。  
インストールを行い、Composer環境の作成、Airflow UIの確認、Composer環境の停止、削除あたりをやってみました。  

# インストール  
インストールは[公式のとおり](https://cloud.google.com/composer/docs/composer-2/run-local-airflow-environments)に行っていきます。  
Githubのソースコードは[ここ](https://github.com/GoogleCloudPlatform/composer-local-dev)にあります。

まずは認証情報の構成を設定します。
```
$ gcloud auth application-default login
$ gcloud auth login
```
ライブラリのインストールを行います。
```
$ git clone https://github.com/GoogleCloudPlatform/composer-local-dev.git
$ cd composer-local-dev
$ pip3 install .
```
ローカル Composer環境の作成を行います。  
GCP上に構築した環境をもとにしてローカルに作成することもできますが、今回はローカル環境で特定のバージョンを指定してexample-local-environmentという名前で構築を行います。
```
$ composer-dev create \
  --from-image-version composer-2.0.32-airflow-2.2.5 \
  example-local-environment
```
--dag-pathを指定しないとcomposerディレクトリが作成されその下にdags, data, airflow.dbなどが作成されます。  
また以下のようにdagのパスを指定するとgit cloneしたcomposer-local-devディレクトリの下ではなくとも環境が作成できます。  
```
/Users/sample/composer-dev-example $ composer-dev create \                         
  --from-image-version composer-2.0.32-airflow-2.2.5 \
  --dags-path dags \
  example-local-environment
```

作成した環境の確認を行います。
```
$ composer-dev list

  Environment Name          │ Version*                      │ State        
╶───────────────────────────┼───────────────────────────────┼─────────────╴
  example-local-environment │ composer-2.0.32-airflow-2.2.5 │ Not started  
```
作成した環境の詳細情報を表示をします。
```
$ composer-dev describe

Composer example-local-environment environment is in state: Not started.

Image version: composer-2.0.32-airflow-2.2.5
Dags directory: /Users/sample/composer-dev-example/dags.
The environment is using credentials from gcloud located at /Users/sample/.config/gcloud.
```
Airflow環境をスタートします。  
Airflow環境が無事にスタートするとlocalhostからUIにアクセスできます。
```
$ composer-dev start example-local-environment
Starting example-local-environment composer environment...
...
Started example-local-environment environment.

1. You can put your DAGs in /Users/sample/composer-dev-example/dags
2. Access Airflow at http://localhost:8080
```
Airflow UIにアクセスを行うと![このように](https://storage.googleapis.com/zenn-user-upload/3be257e7084a-20221225.png)DAGが登録されていることが確認できます。  

DAGを実行するとCLIからログが確認できます。  
```
$ composer-dev logs example-local-environment
...
run_id='scheduled__2022-01-01T00:00:00+00:00', try_number=1) to executor with priority 1 and queue default
2022-12-24T06:45:02.809633465Z [2022-12-24 06:45:02,809] {base_executor.py:85} INFO - Adding to queue: ['airflow', 'tasks', 'run', 'demo', 'airflow', 
'scheduled__2022-01-01T00:00:00+00:00', '--local', '--subdir', 'DAGS_FOLDER/sample.py']
2022-12-24T06:45:02.815494757Z [2022-12-24 06:45:02,814] {sequential_executor.py:59} INFO - Executing command: ['airflow', 'tasks', 'run', 'demo', 'hello', 
'scheduled__2022-01-02T00:00:00+00:00', '--local', '--subdir', 'DAGS_FOLDER/sample.py']
```

DAGの変更を行った場合などにはリスタートを行うと反映されます。  
```
$ composer-dev restart example-local-environment
...
2022-12-24T06:43:38.716401176Z DAG_PROCESSOR_MANAGER_LOG:[2022-12-24 06:43:38,715] {manager.py:518} INFO - Process each file at most once every 30 seconds
2022-12-24T06:43:38.716415718Z [2022-12-24 06:43:38,715] {scheduler_job.py:1235} INFO - Marked 1 SchedulerJob instances as failed
2022-12-24T06:43:38.716975468Z DAG_PROCESSOR_MANAGER_LOG:[2022-12-24 06:43:38,716] {manager.py:519} INFO - Checking for new files in /home/airflow/airflow/dags every 10 seconds
2022-12-24T06:43:38.719036343Z DAG_PROCESSOR_MANAGER_LOG:[2022-12-24 06:43:38,717] {manager.py:704} INFO - Searching for files in /home/airflow/airflow/dags

Started example-local-environment environment.

1. You can put your DAGs in /Users/sample/composer-dev-example/composer-dev-example/dags
2. Access Airflow at http://localhost:8080
```
Airflow 環境の停止を行います。
```
$ composer-dev stop example-local-environment
Stopped composer local environment.
```

Docker イメージの削除..動かない
```
$ docker rmi $(docker images --filter=reference='*/cloud-airflow-releaser/*/*' -q)
Error response from daemon: conflict: unable to delete 428a4887bf60 (must be forced) - image is being used by stopped container 4ccc178d3498
```
Docker イメージの削除..-fオプションで強制削除
```
$ docker rmi -f $(docker images --filter=reference='*/cloud-airflow-releaser/*/*' -q)
Error response from daemon: conflict: unable to delete 428a4887bf60 (must be forced) - image is being used by stopped container 4ccc178d3498
```

# まとめ
公式の通りに行うとインストールから、Airflowのスタート、リスタート、Airflow UIの確認、ログの確認、削除が確認できました。  
裏側で動いているデータベースはSQLite、サービスアカウントの指定はできなさそうなどGCPの環境と完全に同じ状態で動かすことはできなそうですが、実際のGCPの環境と同じイメージで動かせることやAirflowを動かす環境を自分で作成しなくてもよい（Airflowを動かすにはwebserver、scheduler、dbを動かす必要がありそこが[セットアップファイル](https://github.com/GoogleCloudPlatform/composer-local-dev/blob/2c8a516c547125c7ebf9e8e120b8301bb5098763/composer_local_dev/docker_files/entrypoint.sh)）になって自分達で管理する必要がないのも便利そうです。  

