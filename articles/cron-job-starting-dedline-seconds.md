---
title: "Cron JobでstartingDeadlineSeconds設定のすすめ"
emoji: "🦥"
type: "tech"
topics: ["kubernetes", "cronjob", "gcp"]
published: true
---

# はじめに
Kubernetesでジョブをスケジュール実行する際に使用することができるCronJobには設定できるオプションがいくつかあります。  
今回はその中でもstartingDeadlineSecondsというオプションを設定しなかったため起こった動作を踏まえて説明を行いたいと思います。

# 説明しないこと
Kubernetes、Jobの説明、インストール方法などの説明は今回は省略します。

# 設定しなかったので何が起こったか
GCP上でCronJobを実務で使用する際に何らかの都合でJobの実行を一時的に止めるためにCloud Consoleの画面からCronJobを停止を行い、問題が解決したために再度Jobのスケジュールを再開するために再び画面から再開を押すことがあるかと思います。  
その場合に今回のstaringDeadlineSecondsを設定していなかったため、スケジュールで指定していた時間ではないにも関わらずCronJobを再開したタイミングでJobが実行してしまいました。  
本来想定していた動きとしては、CronJobの再開時にはJobは実行されず、次のスケジュールしてあるタイミングでJobが起動するというものでした。  

# なぜ再開のタイミングでJobが起動したか
今回の想定外のJobの起動が起こった要因は停止状態（suspend）のJobの実行ステータスの扱いが関係しています。  
CronJobでは停止状態でもスケジュールされた時間にJobの起動が行えないと、内部的には失敗として記録されます。  
そのためCronJobは再開された際に失敗として記録されていたJobを実行する動きをします。

# staringDeadlineSecondsを設定すると
では停止状態のJobを起動したタイミングで動かさないようするにはどうしたら良いでしょうか。  
ここでstaringDeadlineSecondsを設定することで防ぐことができます。  
startingDeadlineSecondsとは何らかの理由でJobが実行できなかった場合に何秒後までなら起動を行うかを指定するための設定です。  
startingDeadlineSecondsを30分で設定すると、スケジュールされた時間を過ぎても30分以内であればJobが実行されますが、30分をすぎると実行が行われなくなります。  
そのため停止状態のJobが失敗として記録されるが、適切にstartingDeadlineSecondsを設定することで、再実行を行う期間をすぎるため再開時に起動が行われなくなります。  
ここで適切にと書いたのは、当たり前ですが停止を行ってからstartingDeadlineSecondsの範囲内で再開を行うとJobは起動されますので注意が必要です。  
またもうひとつの注意点としては公式にも記載がありますが、10秒以下に設定をするとそもそもCronJobがスケジュールをされなくなります。([Kubernetes/CronJob limitations](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#cron-job-limitations))  
他の注意点としてはJobが100回連続失敗した際の挙動とも関連してくる部分もありますが、そこはMercariさんの記事に詳しく書かれているので参照していただけるといいと思います。  参考: [Kubernetes CronJobと仲良くなりたい](https://engineering.mercari.com/blog/entry/k8s-cronjob-20200908/)
