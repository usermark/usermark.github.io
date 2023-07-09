---
layout: article
title: '[Hexo] 用Hexo + Github Pages架設個人網誌'
date: 2023-07-09 15:32:03
tags:
- Hexo
- Github Pages
---
原本是用Jekyll + Github Pages架設個人網誌，轉移到Hexo後發現真得很方便，這次便將安裝過程紀錄下來。
<!--more-->
# 安裝Hexo

安裝前需要有Node.js和Git。

輸入下列指令安裝
```sh
npm install hexo-cli -g
```

安裝完後，可輸入hexo version查看Hexo版本和相關資訊如下：
```
hexo-cli: 4.3.1
os: win32 10.0.19045
node: 18.16.1
acorn: 8.8.2
ada: 1.0.4
ares: 1.19.1
brotli: 1.0.9
cldr: 42.0
icu: 72.1
llhttp: 6.0.11
modules: 108
napi: 8
nghttp2: 1.52.0
nghttp3: 0.7.0
ngtcp2: 0.8.1
openssl: 3.0.9+quic
simdutf: 3.2.2
tz: 2022g
undici: 5.21.0
unicode: 15.0
uv: 1.44.2
uvwasi: 0.0.15
v8: 10.2.154.26-node.26
zlib: 1.2.13
```

# 初始化Hexo

輸入下列指令初始化
```sh
hexo init <資料夾名稱>
```

結果
```
INFO  Cloning hexo-starter https://github.com/hexojs/hexo-starter.git
[32mINFO [39m Install dependencies
INFO  Start blogging with Hexo!
```

切換到剛才建立的資料夾後，安裝所需的npm套件
```
cd <資料夾名稱>
npm install
```

# Hexo相關指令

新增文章
```sh
hexo new [layout] <title>
```

清除靜態檔和快取
```sh
hexo cl
```

產生靜態檔
```sh
hexo g
```

啟動伺服器
```sh
hexo server
```

結果
```
INFO  Validating config
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/ . Press Ctrl+C to stop.
```

# 切換主題為NexT

到這邊先別急著走，推薦一套超好用的主題NexT，裡面已彙整好常用的個人網誌相關套件，只要做幾個設定便可運行。

輸入指令clone NexT主題
```sh
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

編輯_config.yml切換主題
```yaml
theme: next
```

複製一份themes\next\_config.yml至source\_data\next.yml，基於版本控管不處理themes的規則，可用複製出來的next.yml做設定，後面會用到。

重新執行hexo g和hexo server便可看到新的主題。

# 新增頁面：關於

點擊網誌上的關於，會發現是空的，此頁面需要手動建立。

輸入下列指令，會於source底下新增about\index.md
```sh
hexo new page about
```

編輯about\index.md，習慣會將留言關閉
```yaml
---
title: 關於
date: 2023-07-08 16:52:20
comments: false #關閉留言
---
```

# 新增頁面：標籤

輸入下列指令，會於source底下新增tags\index.md
```sh
hexo new page tags
```

編輯tags\index.md
```yaml
---
title: 標籤
date: 2023-07-08 20:56:45
type: tags #必須加上此行宣告
comments: false #關閉留言
---
```

編輯source\_data\next.yml，分別打開about和tags
```yaml
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  #categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

# 新增功能：Gitalk

新增留言功能，順便說明如何打開相關套件。

首先至Github的Settings > Developer Settings > OAuth Apps > New OAuth App，輸入相關資訊。

申請完後會獲得一組Client ID和Client Secrt。

編輯source\_data\next.yml
```yaml
gitalk:
  enable: true
  github_id: usermark # GitHub repo owner
  repo: usermark.github.io # Repository name to store issues
  client_id: xxx # GitHub Application Client ID
  client_secret: xxx # GitHub Application Client Secret
  admin_user: usermark # GitHub repo owner and collaborators, only these guys can initialize gitHub issues
  distraction_free_mode: true # Facebook-like distraction free mode
  # Gitalk's display language depends on user's browser or system environment
  # If you want everyone visiting your site to see a uniform language, you can set a force language value
  # Available values: en | es-ES | fr | ru | zh-CN | zh-TW
  language: zh-TW
```

輸入npm指令安裝gitalk套件
```sh
npm i --save gitalk
```

即可在每篇文章的最後看到留言功能。注意要部署到server才可正常登入使用，本地localhost無法使用。

# 部署到Github Pages

安裝Git相關套件
```sh
npm install hexo-deployer-git --save
```

編輯_config.yml，建議使用gh-pages分支，而不是master分支，方便將原始檔和實際要公開部署的分開控管。
```yaml
deploy:
  type: git
  repo: https://github.com/username/username.github.io.git
  branch: gh-pages
```

輸入下列指令部署，第一次跑會要求登入Github
```sh
hexo cl
hexo g -d
```

完成後，便可在個人頁面 https://username.github.io 查看部署結果。

**參考資料**
1. [【學習筆記】如何使用 Hexo + GitHub Pages 架設個人網誌](https://hackmd.io/@Heidi-Liu/note-hexo-github)
2. [十分鐘超詳細替 Hexo Next 開啟 Gitalk 留言版](https://israynotarray.com/hexo/20191206/2397475810/)