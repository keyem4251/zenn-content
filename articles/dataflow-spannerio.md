---
title: "Cloud Dataflow SpannerIOのwithGroupingFactor"
emoji: "🥁"
type: "tech"
topics: ["gcp", "dataflow", "beam"]
published: false
---

# はじめに
Cloud DataflowはApache BeamをGCP上で実行するための[サービス](https://cloud.google.com/dataflow?hl=ja)です。  
フルマネージドになっており、ワーカーの水平自動スケーリングの機能を持っています。  
Java、Pythonなどのコードを用いてBigQueryなどのデータを変換しSpannerなどの他のデータストアに書き込むことができます。  
Cloud Dataflowは実行時に様々なパラメータを設定できるので、今回はその中で2つ紹介をしたいと思います。  
