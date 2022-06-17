---
title: "Cloud Dataflowの監視、ワーカー数のパラメータ設定"
emoji: "🥁"
type: "tech"
topics: ["gcp", "dataflow", "beam"]
published: true
---

# はじめに
Cloud DataflowはApache BeamをGCP上で実行するための[サービス](https://cloud.google.com/dataflow?hl=ja)です。  
フルマネージドになっており、ワーカーの水平自動スケーリングの機能を持っています。  
Java、Pythonなどのコードを用いてBigQueryなどのデータを変換しSpannerなどの他のデータストアに書き込むことができます。  
Cloud Dataflowは実行時に様々なパラメータを設定できるので、今回はその中で2つ紹介をしたいと思います。  

# enable_stackdriver_agent_metrics
Cloud DataflowはもともとGCPのマネージドサービスのため何も設定を行わなくてもジョブの失敗、実行時間などのメトリクスが設定されています。  
そして、そのメトリクスをもとにアラートポリシーやダッシュボードなどで通知、監視が行えるようになっています。  
しかし、既存の設定のままでは実際にデータを処理しているワーカー（VMインスタンス）の情報を取得することができません。  
またDataflowは水平自動スケーリングは行いますが、各ワーカーの垂直方向のスケーリングは行わないためメモリなどの監視を行わないとメモリが足りなくなりOOMが発生し、ジョブが失敗することがあります。  
そこで実行時に[`--experiments=enable_stackdriver_agent_metrics`を設定する](https://cloud.google.com/dataflow/docs/guides/using-cloud-monitoring?hl=ja#receive_worker_vm_metrics_from_the_agent)ことでワーカーのCPU、メモリなどのメトリクスを取得できます。  
そこからメトリクスをもとに監視やダッシュボードで確認を行うことができます。  
:::message  
experimentsのパラメータとなっているためプロダクション環境のサービスへの適用はご注意ください。
:::

# maxNumWorkers
Dataflowは水平自動スケーリングを行い入力のデータサイズなどをもとに自動でワーカー数を調整してくれます。  
しかしあくまでDataflow内の処理を最適化するだけで、書き込み先のデータストア（SpannerやBigTable）のCPUの使用率を考慮したスケーリングは行いません。  
そのためDataflowが判断したワーカー数が書き込み先のデータストアのノード数では処理が間に合わないとCPUの使用率が高くなり、データストアからの読み取りが遅くなるなどの影響が出る可能性があります。  
これを回避するためにワーカーの最大数を書き込み先のデータストアのノード数などCPU使用率を確認しつつ調整する必要があります。  
設定方法は[`--maxNumWorkers`に数字を設定する](https://cloud.google.com/dataflow/docs/guides/deploying-a-pipeline?hl=ja#autotuning-features)ことでできます。

# ちなみにDataflow Primeというのもあります  
先ほど既存のDataflowでは垂直方向のスケーリングは行わないとありましたが、Dataflow Primeでは垂直方向の自動スケーリングも行ってくれます。  
まだプレビューのようなので早くGAになって欲しいですね。
[Dataflow Prime](https://cloud.google.com/dataflow/docs/guides/enable-dataflow-prime?hl=ja)
