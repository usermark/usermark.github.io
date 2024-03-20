---
title: '[GCP] 上傳映像檔至 Artifact Registry'
date: 2024-03-21 02:19:08
tags:
- GCP
- Docker
---
參考 https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling 即可，也記錄自己理解的。
<!--more-->
首先決定映像檔要放那個[地區](https://cloud.google.com/artifact-registry/docs/repositories/repo-locations)，例如asia-east1(台灣)，
確認 %USERPROFILE%\.docker\config.json 裡面是否有 asia-east1-docker.pkg.dev
第一次使用不會有，要輸入下列指令新增
```sh
gcloud auth configure-docker asia-east1-docker.pkg.dev
```

新增後長這樣
```json
{
    "auths": {
        "asia-east1-docker.pkg.dev": {},
        "https://index.docker.io/v1/": {}
    },
    "credsStore": "desktop",
    "credHelpers": {
        "asia-east1-docker.pkg.dev": "gcloud"
    },
    "currentContext": "default"
}
```

進入GCP頁面 > IAM 與管理 > 服務帳戶，建立服務帳戶，並授予"Artifact Registry 服務代理"的角色。

建立憑證並登入docker
```
gcloud auth print-access-token --impersonate-service-account docker@XXX.iam.gserviceaccount.com | docker login -u oauth2accesstoken --password-stdin https://asia-east1-docker.pkg.dev
```