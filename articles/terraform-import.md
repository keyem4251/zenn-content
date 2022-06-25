---
title: "terraform importの小ネタ"
emoji: "🎨"
type: "tech"
topics: ["terraform", "gcp"]
published: true
---

# はじめに
クラウド環境で手動で行った設定を後からterraformに取り込むことがあると思います。  
その際に`terraform import`でリソースを取り込むことができます。  
今回は`terraform import`で最近気づいた小ネタを紹介したいと思います。  

# 話さないこと
* terraformについて
* terraformerについて（terraform importを使わずに構成を取り込むツール）

# terraform import
terraformでリストの構造を使用することがあると思います。  
具体的にはGCPのIAMのリソースに対して複数のロールを割り当ててtfstateが下記のようになる場合です。  
```
resource "google_project_iam_member" "project[0]" {
  project = "your-project-id"
  role    = "roles/editor"
  member  = "user:jane@example.com"
}

resource "google_project_iam_member" "project[1]" {
  project = "your-project-id"
  role    = "roles/viwer"
  member  = "user:jane@example.com"
}
```

このIAMに手動で追加したロールをterraformに取り込む際には以下だとエラーが出ます。  
```
$ terraform import google_project_iam_member.my_project[2] "your-project-id roles/admin user:foo@example.com"
```

下記だとエラーは出ませんが`google_project_iam_member.my_project[2]`ではなく`google_project_iam_member.my_project`に対してのtfstateが新しく作成されてしまいます。
```
$ terraform import google_project_iam_member.my_project "your-project-id roles/admin user:foo@example.com"
```

これを解決するには以下のようにダブルクォーテーションでリソースを囲みます。  
```
$ terraform import "google_project_iam_member.my_project[2]" "your-project-id roles/admin user:foo@example.com"
```

以上小ネタでした。  
