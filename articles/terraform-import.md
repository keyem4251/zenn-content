---
title: "terraform importã®å°ãã¿"
emoji: "ð¨"
type: "tech"
topics: ["terraform", "gcp"]
published: true
---

# ã¯ããã«
ã¯ã©ã¦ãç°å¢ã§æåã§è¡ã£ãè¨­å®ãå¾ããterraformã«åãè¾¼ããã¨ãããã¨æãã¾ãã  
ãã®éã«`terraform import`ã§ãªã½ã¼ã¹ãåãè¾¼ããã¨ãã§ãã¾ãã  
ä»åã¯`terraform import`ã§æè¿æ°ã¥ããå°ãã¿ãç´¹ä»ãããã¨æãã¾ãã  

# è©±ããªããã¨
* terraformã«ã¤ãã¦
* terraformerã«ã¤ãã¦ï¼terraform importãä½¿ããã«æ§æãåãè¾¼ããã¼ã«ï¼

# terraform import
terraformã§ãªã¹ãã®æ§é ãä½¿ç¨ãããã¨ãããã¨æãã¾ãã  
å·ä½çã«ã¯GCPã®IAMã®ãªã½ã¼ã¹ã«å¯¾ãã¦è¤æ°ã®ã­ã¼ã«ãå²ãå½ã¦ã¦tfstateãä¸è¨ã®ããã«ãªãå ´åã§ãã  
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

ãã®IAMã«æåã§è¿½å ããã­ã¼ã«ãterraformã«åãè¾¼ãéã«ã¯ä»¥ä¸ã ã¨ã¨ã©ã¼ãåºã¾ãã  
```
$ terraform import google_project_iam_member.my_project[2] "your-project-id roles/admin user:foo@example.com"
```

ä¸è¨ã ã¨ã¨ã©ã¼ã¯åºã¾ããã`google_project_iam_member.my_project[2]`ã§ã¯ãªã`google_project_iam_member.my_project`ã«å¯¾ãã¦ã®tfstateãæ°ããä½æããã¦ãã¾ãã¾ãã
```
$ terraform import google_project_iam_member.my_project "your-project-id roles/admin user:foo@example.com"
```

ãããè§£æ±ºããã«ã¯ä»¥ä¸ã®ããã«ããã«ã¯ã©ã¼ãã¼ã·ã§ã³ã§ãªã½ã¼ã¹ãå²ã¿ã¾ãã  
```
$ terraform import "google_project_iam_member.my_project[2]" "your-project-id roles/admin user:foo@example.com"
```

ä»¥ä¸å°ãã¿ã§ããã  
