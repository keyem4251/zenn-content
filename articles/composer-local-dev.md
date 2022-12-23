---
title: "Cloud Composerのローカル開発 composer-local-dev お試し"
emoji: ""
type: "tech"
topics: ["airflow", "GCP", "Composer"]
published: false
---
# はじめに

# composer-dev-example
[source code](https://github.com/GoogleCloudPlatform/composer-local-dev)
[document](https://cloud.google.com/composer/docs/composer-2/run-local-airflow-environments)

認証情報の構成
```
$ gcloud auth application-default login
$ gcloud auth login
```
ライブラリのInstall
```
$ cd composer-local-dev
$ pip3 install .
```
ローカル Composer環境の作成
```
$ composer-dev create \
  --from-image-version IMAGE_VERSION \
  --dags-path LOCAL_DAGS_PATH \
  LOCAL_ENVIRONMENT_NAME
```
例
```
$ composer-dev create \
  --from-image-version composer-2.0.32-airflow-2.2.5 \
  example-local-environment
```
--dag-pathを指定しないとcomposerディレクトリが作成されその下にdags, data, airflow.dbなどが作成される。  

作成した環境の確認
```
$ composer-dev list

  Environment Name          │ Version*                      │ State        
╶───────────────────────────┼───────────────────────────────┼─────────────╴
  example-local-environment │ composer-2.0.32-airflow-2.2.5 │ Not started  
```
作成した環境の詳細情報を表示
```
$ composer-dev describe

Composer example-local-environment environment is in state: Not started.

Image version: composer-2.0.32-airflow-2.2.5
Dags directory: /Users/sample/composer-dev-example/composer-local-dev/composer/example-local-environment/dags.
The environment is using credentials from gcloud located at /Users/sample/.config/gcloud.
```
Airflow 環境をスタート（localhost:8080でAirflow UIにアクセス）
```
$ composer-dev start example-local-environment
Starting example-local-environment composer environment...
...
Started example-local-environment environment.

1. You can put your DAGs in /Users/sample/composer-dev-example/composer-local-dev/composer/example-local-environment/dags
2. Access Airflow at http://localhost:8080
```
Airflow 環境の停止
```
$ composer-dev stop example-local-environment
Stopped composer local environment.
```
Airflow 環境の削除..動かない
```
$ composer-dev remove example-local-environment
Usage: composer-dev [OPTIONS] COMMAND [ARGS]...
Try 'composer-dev --help' for help.
╭─ Error ────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ No such command 'remove'.                                                                                                      │
╰────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
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
