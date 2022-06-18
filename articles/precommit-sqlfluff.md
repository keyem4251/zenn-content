---
title: "SQLFluffをpre-commitで自動化"
emoji: "👽"
type: "tech"
topics: ["precommit", "sql", "sqlfluff", "linter", "bigquery"]
published: true
---

# はじめに
SQLのLintツールにSQLFluffというものがあります、コミット時に自動でLintをかけるためのpre-commitと組み合わせて実行させるための手順をまとめました。  

# pre-commitインストール&設定方法
pre-commitの公式の説明、手順は[こちら](https://pre-commit.com)。  
pipでインストールできます。
```
$ pip install pre-commit
$ pre-commit --version
pre-commit 2.19.0
```
設定ファイルを`.pre-commit-config.yaml`で作成します。  
ファイルの内容は以下のように設定します。  
```
repos:
- repo: https://github.com/sqlfluff/sqlfluff
  rev: 0.13.2
  hooks:
    - id: sqlfluff-lint
      args: [--dialect, bigquery]
    - id: sqlfluff-fix
      args: [--dialect, bigquery]
```
ファイルを作成したら設定をインストールします。  
```
$ pre-commit install
pre-commit installed at .git/hooks/pre-commit
```
これでpre-commitの設定完了となります。  

# SQLFluffインストール&設定方法
SQLFluffの公式の説明、手順は[こちら](https://www.sqlfluff.com)。
こちらも同じくpipでインストールできます。  
```
$ pip install sqlfluff
$ sqlfluff version
0.13.2
```
SQLFLuffの設定はひとまず完了です。  

# 動作確認
サンプルのファイルとして以下のようなSQLファイルを作成します。  
```
SELECT a+b  AS foo,
c AS bar from my_table
```
これをコミットして実際にLinterが実行されるか確認します。（初回実行時には環境のインストールが走るため時間がかかります。）  
```
$ git add sample.sql
$ git commit -m "sample file commit"
[INFO] Initializing environment for https://github.com/sqlfluff/sqlfluff.
[INFO] Installing environment for https://github.com/sqlfluff/sqlfluff.
[INFO] Once installed this environment will be reused.
[INFO] This may take a few minutes...
sqlfluff-lint............................................................Failed
- hook id: sqlfluff-lint
- exit code: 65

== [sample.sql] FAIL                     
L:   1 | P:   1 | L034 | Select wildcards then simple targets before calculations
                       | and aggregates.
L:   1 | P:   1 | L036 | Select targets should be on a new line unless there is
                       | only one select target.
L:   1 | P:   9 | L006 | Missing whitespace before +
L:   1 | P:   9 | L006 | Missing whitespace after +
L:   1 | P:  11 | L039 | Unnecessary whitespace found.
L:   2 | P:   1 | L003 | Expected 1 indentations, found 0 [compared to line 01]
L:   2 | P:  10 | L010 | Keywords must be consistently upper case.
All Finished 📜 🎉!


sqlfluff-fix.............................................................Failed
- hook id: sqlfluff-fix
- files were modified by this hook

==== finding fixable violations ====
== [sample.sql] FAIL                           
L:   1 | P:   1 | L034 | Select wildcards then simple targets before calculations
                       | and aggregates.
L:   1 | P:   1 | L036 | Select targets should be on a new line unless there is
                       | only one select target.
L:   1 | P:   9 | L006 | Missing whitespace before +
L:   1 | P:   9 | L006 | Missing whitespace after +
L:   1 | P:  11 | L039 | Unnecessary whitespace found.
L:   2 | P:   1 | L003 | Expected 1 indentations, found 0 [compared to line 01]
L:   2 | P:  10 | L010 | Keywords must be consistently upper case.
==== fixing violations ====
7 fixable linting violations found
FORCE MODE: Attempting fixes...
Persisting Changes...
== [sample.sql] PASS
Done. Please check your files to confirm.
```
これでファイルを確認すると以下のようにフィールドの順番、fromが大文字に変換、FROM句が改行されておりファイルが修正されています。
```
SELECT
    c AS bar,
    a + b AS foo
FROM my_table
```
いい感じに修正されているのでこの状態で再度コミットします。
```
$ git add .
$ git commit -m "commit"
sqlfluff-lint............................................................Passed
sqlfluff-fix.............................................................Passed
[main a5dac82] commit
 3 files changed, 13 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 .pre-commit-config.yaml
 create mode 100644 sample.sql
```
無事にコミットが成功しました。  
今回は基本の設定にしましたが、SQLFluff自体にも設定ファイルを作成することで独自のルールを設定することができます。  
設定方法は[こちら](https://docs.sqlfluff.com/en/stable/gettingstarted.html#custom-usage)をご覧ください。
