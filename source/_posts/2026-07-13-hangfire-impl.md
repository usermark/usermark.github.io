---
title: "[Hangfire] 入門 + SQLite"
date: 2026-07-13 15:10:32
tags:
- Hangfire
- .NET Framework
- SQLite
---
公司還在使用工作排程器，隨著案子和排程越來越多，變得不好控管，若遇到狀況，還要遠端登入主機查看，或是重啟排程。
才想來導入 Hangfire，它還有提供美美的網頁後台，省下登入主機查看的流程。
<!--more-->

# 安裝 Hangfire

使用 NuGet 套件管理員安裝以下套件
- Hangfire
- Hangfire.AspNet
- Hangfire.SQLite
- Microsoft.Owin

修改 Startup.cs
```cs
using Hangfire;
using Hangfire.SQLite;
using Microsoft.Owin;
using Owin;
using System;
using System.Collections.Generic;
using System.Web.Hosting;

[assembly: OwinStartup(typeof(ETL.Startup))]
namespace ETL
{
    public class Startup
    {
        private static readonly string SqliteDbPath = HostingEnvironment.MapPath("~/App_Data/Hangfire.sqlite");

        public static IEnumerable<IDisposable> GetHangfireConfiguration()
        {
            GlobalConfiguration.Configuration
                .SetDataCompatibilityLevel(CompatibilityLevel.Version_180)
                .UseSimpleAssemblyNameTypeSerializer()
                .UseRecommendedSerializerSettings()
                .UseSQLiteStorage($"Data Source={SqliteDbPath};");

            GlobalJobFilters.Filters.Add(new AutomaticRetryAttribute { Attempts = 0 });

            // https://github.com/wanlitao/HangfireExtension/issues/3
            yield return new BackgroundJobServer(new BackgroundJobServerOptions { WorkerCount = 1 });
        }

        public void Configuration(IAppBuilder app)
        {
            app.UseHangfireAspNet(GetHangfireConfiguration);
            app.UseHangfireDashboard();

            // 工作排程寫在這下面
            MyTask.Setup();
        }
    }
}
```

第 14 行將 sqlite 檔放 App_Data 資料夾底下，不會被第三方取得。
第 24 行表示排程失敗的話，不會進行重試。
第 27 行是修正 Hangfire.SQLite 套件的 Bug，Hangfire 預設會開 20 條 Worker Thread，會出現同個排程一次同時跑 20 次，這裡先將 Worker Thread 改成 1 條來修正。
弟 33 行啟用網頁後台。

新增 MyTask.cs
```cs
using Hangfire;
using System;
using System.Configuration;

namespace ETL
{
    public class MyTask
    {
        public static void Setup()
        {
            RecurringJob.AddOrUpdate("我是排程", () => Run(), Cron.Daily(8), new RecurringJobOptions
            {
                TimeZone = TimeZoneInfo.Local
            });
        }

        public static void Run()
        {
            // do something
        }
    }
}
```

第 11 行表示依當地時間，固定每天早上8點執行。

# 確保網站永遠處於執行狀態

## 應用程式初始化

確認已安裝「應用程式初始化」(ApplicationInitializationModule) 模組

![](/assets/hangfire1.png)

如果還沒有，到伺服器管理員安裝

![](/assets/hangfire2.png)

## IIS 相關設定

應用程式集區 >「進階設定」>「啟動模式」設「AlwaysRunning」和「閒置逾時」設 0

![](/assets/hangfire3.png)

站台 >「進階設定」啟用「預先載入已啟用」

![](/assets/hangfire4.png)

## 新增 ApplicationPreload.cs

Hangfire.AspNet 為官方出的套件，用來簡化讓服務可以始終運行的實作。

```cs
using Hangfire;
using System.Web.Hosting;

namespace ETL
{
    public class ApplicationPreload : IProcessHostPreloadClient
    {
        public void Preload(string[] parameters)
        {
            HangfireAspNet.Use(Startup.GetHangfireConfiguration);
        }
    }
}
```

**參考資料**

1. [Hangfire 筆記1 – 使用 SQLite](https://blog.darkthread.net/blog/hangfire-sqlite/)
2. [Hangfire 筆記2 - 執行定期排程](https://blog.darkthread.net/blog/hangfire-recurringjob-notes/)
