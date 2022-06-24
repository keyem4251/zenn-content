---
title: "Kotlinで自作DBMS"
emoji: "🪐"
type: "tech"
topics: ["kotlin", "dbms", "db", "database"]
published: true
---
# はじめに
データベースの勉強のために[SimpleDB](http://www.cs.bc.edu/~sciore/simpledb/) というJavaで書かれたデータベースの実装が書かれている書籍を参考にKotlinで実装を行いました。  

# どんなものができるか
IntelliJを用いてKotlinを実行してます。コードは[こちらのGitHub](https://github.com/keyem4251/kotlin-dbms/blob/master/README.md) にあります。

### データベース作成
kotlin/simpledb/tools/CreateStudentDB.ktをIntelliJから実行すると、指定したディレクトリがない場合にデータベースの作成、テーブルの作成、データのinsertを行います。  
![データベース作成](https://storage.googleapis.com/zenn-user-upload/def2c002fdf8-20220326.png)

### データベース起動
kotlin/simpledb/server/StartServer.ktをIntelliJから実行すると、データベースへの接続を求められるので、接続先を入力します。  
作成済みのstudentテーブルをselectします。
![データベースに接続、select](https://storage.googleapis.com/zenn-user-upload/c47053527712-20220326.png)

whereを用いた条件、他のテーブルとのjoinも行えます。
![whereで条件を満たす行を抽出、他のテーブルとJOIN](https://storage.googleapis.com/zenn-user-upload/f159c4e63767-20220326.png)

:::message  
GitHub上には動画が載っているので実際の動いている様子を見たい方はGitHubを参照ください  
再掲[GitHub](https://github.com/keyem4251/kotlin-dbms/blob/master/README.md)
:::

# 本の概要
書籍の内容としては全部で15章となっており目次は下記となっています。  

|章番号|タイトル|  
|----|----|
|1|Database Systems|
|2|JDBC|
|3|Disk and File Management|
|4|Memory Management|
|5|Transaction Management|
|6|Record Management|
|7|Metadata Management|
|8|Query Processing|
|9|Parsing|
|10|Planning|
|11|JDBC interfaces|
|12|Indexing|
|13|Materialization and Sorting|
|14|Effective Buffer Utilization|
|15|Query Optimization|

それぞれの章でタイトルの内容の解説が書いてあるだけでなく、FileManager、BufferManager、Plan、Parserなどのクラスの実装が載っているため実際にコードにするとどうなるかを見ながらデータベースの理論について学ぶことができます。  
最後まで実装を終えると以下のような内容が実行できるようになります。  
- select F1, F2 from T1 where F2 = Value
- select F1, F2 from T1, T2 where F3=F4
- insert into T(F1, F2, F3) values ('a', 'b', 'c')
- delete from T where F1=F2
- update T set F1='a' where F1=F2
- create table T(F1 int, F2 varchar(9))
- create view V as select F1, F2 from T1, T2 where F3=F4
- create index I on T(F)

:::message  
SimpleDBでは文字列と数値型のみ対応をしています  
11章のJDBCを用いた実装は書籍の中ではJavaのRMIを用いた実装も載っていますが、今回はEmbedded JDBC interfaceしか実装してません  
:::

# まとめ
データベースについてどういった基本的な処理が後ろで動いているか想像ができるようになるという点もそうですが、単純に自分で書いたコードでテーブル、インデックスが作れたり、テーブル同士のJOINまでできるのには感動しました。  
書籍の中で基本的な実装に加えて、各章の最後には拡張課題が載っているのですが今回はそこには取り組んでいないので、また書籍を見返しつつ拡張課題にも取り組めたらなと思います。
