---
title: '[GCP] GCP上傳映像檔至 Artifact Registry'
date: 2024-03-21 02:19:08
tags:
- GCP
- Docker
---

參考 https://cloud.google.com/artifact-registry/docs/docker/pushing-and-pulling 即可，也記錄自己理解的。
<!--more-->
# 建立存放區

首先決定映像檔要放那個[地區](https://cloud.google.com/artifact-registry/docs/repositories/repo-locations)，例如asia-east1(台灣)。於存放區建立
![](/assets/gcp_hub.png)

# 設定docker

確認 %USERPROFILE%\\.docker\config.json 裡面是否有 asia-east1-docker.pkg.dev
第一次使用不會有，會長這樣
```json
{
    "auths": {},
    "credsStore": "desktop",
    "currentContext": "default"
}
```

要輸入下列指令新增
```sh
gcloud auth configure-docker asia-east1-docker.pkg.dev
```

會詢問是否新增，輸入Y後按Enter
```
Adding credentials for: asia-east1-docker.pkg.dev
After update, the following will be written to your Docker config file located at [C:\Users\XXX\.docker\config.json]:
 {
  "credHelpers": {
    "asia-east1-docker.pkg.dev": "gcloud"
  }
}

Do you want to continue (Y/n)?
```

新增後長這樣
```json
{
    "auths": {},
    "credsStore": "desktop",
    "currentContext": "default",
    "credHelpers": {
        "asia-east1-docker.pkg.dev": "gcloud"
    }
}
```

進入GCP頁面 > IAM 與管理 > 服務帳戶，建立服務帳戶，並授予"Artifact Registry 服務代理"的角色。
![](/assets/gcp_role_auth.png)

使用剛新增好的服務帳戶，建立憑證並登入docker
```
gcloud auth print-access-token --impersonate-service-account XXX@XXX.iam.gserviceaccount.com | docker login -u oauth2accesstoken --password-stdin https://asia-east1-docker.pkg.dev
```

再次查看 %USERPROFILE%\\.docker\config.json，發現auths欄位有新增值。
```json
{
    "auths": {
        "asia-east1-docker.pkg.dev": {}
    },
    "credsStore": "desktop",
    "credHelpers": {
        "asia-east1-docker.pkg.dev": "gcloud"
    },
    "currentContext": "default"
}
```

# 建立tag並上傳映像檔

透過指令先查看全部的image
```sh
docker images
```

結果
```
REPOSITORY           TAG       IMAGE ID       CREATED       SIZE
snowdreamtech/frps   latest    10dcc047cf37   4 weeks ago   25.7MB
snowdreamtech/frpc   latest    9c3e52b55b77   4 weeks ago   21.8MB
```

輸入指令下tag
```sh
docker tag snowdreamtech/frps asia-east1-docker.pkg.dev/xxx/hub/frps:v1
```
其中asia-east1-docker.pkg.dev/xxx/hub便是對應存放區
![](/assets/gcp_repo.png)

再次輸入docker images指令查看，會多一個image
```
REPOSITORY                                           TAG       IMAGE ID       CREATED       SIZE
snowdreamtech/frps                                   latest    10dcc047cf37   4 weeks ago   25.7MB
asia-east1-docker.pkg.dev/xxx/hub/frps               v1        10dcc047cf37   4 weeks ago   25.7MB
snowdreamtech/frpc                                   latest    9c3e52b55b77   4 weeks ago   21.8MB
```

輸入指令上傳image
```sh
docker push asia-east1-docker.pkg.dev/xxx/hub/frps:v1
```

```
The push refers to repository [asia-east1-docker.pkg.dev/xxx/hub/frps]
d51ae83e6fc9: Pushed
aedc3bda2944: Pushed
v1: digest: sha256:a644c27fa4619837523ec9c8749953edbb5d4a129e49b95cc9371bf9c24c876f size: 739
```

上傳完後，便可在存放區看到剛上傳的image囉
![](/assets/gcp_repo2.png)