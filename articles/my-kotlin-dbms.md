---
title: "Kotlinで自作DBMS"
type: "tech"
topics: ["kotlin", "dbms"]
published: false
---
# はじめに
データベースの勉強のために[SimpleDB](http://www.cs.bc.edu/~sciore/simpledb/) というJavaで書かれたデータベースの実装が書かれている書籍を参考にKotlinで実装を行いました。  

# どんなものができるか
IntelliJを用いてKotlinを実行してます。コードは[こちらのGithub](https://github.com/keyem4251/kotlin-dbms/blob/master/README.md) にあります。

### データベース作成
https://user-images.githubusercontent.com/53423344/160232120-e0013c3e-f3c1-41c3-a733-77cbaf981689.mp4

### データベース起動
https://user-images.githubusercontent.com/53423344/160232133-1e9ac9e4-c1dc-40f4-a427-9f0aeae682fd.mp4

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
11章のJDBCを用いた実装は書籍の中ではJavaのRMIを用いた実装も載っていますが、今回はEmbedded JDBC interfaceしか実装してません  
:::

# まとめ
データベースについてどういった基本的な処理が後ろで動いているか想像ができるようになるという点そうですが、単純に自分で書いたコードでテーブル、インデックスが作れたり、テーブル同士のJOINまでできるのには感動しました。  
書籍の中で基本的な実装に加えて、各章の最後には拡張課題が載っているのですが今回はそこには取り組んでいないので、また書籍を見返しつつ拡張課題にも取り組めたらなと思います。
