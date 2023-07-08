---
layout: article
title: '[Python] Redmine API'
date: 2023-04-03 18:04
tags: Python
---
<!--more-->

參考<https://www.redmine.org/projects/redmine/wiki/rest_api>

要使用redmine api，header需加上X-Redmine-API-Key，API存取金鑰要從"我的帳戶"取得。
![](/assets/redmine_profile.png)

# 管理議題

Redmine API的議題提供json檢視，用GET取得議題單內容，用PUT更新議題內容和狀態。

```python
import requests


requests.packages.urllib3.disable_warnings()  # 關閉InsecureRequestWarning
url = 'https://$redmine_url'
api_key = '$api_key'

def get_issue(issue_no):
    res = requests.get(f'{url}{issue_no}.json?include=journals', 
                       verify=False, 
                       headers={'X-Redmine-API-Key': api_key})
    if res.status_code != requests.codes.ok:
        print('失敗', res.status_code)
        return
    issue = res.json()['issue']
    print('主旨:', issue['subject'])
    print('狀態:', issue['status']['name'])
    assigned_to_name = None
    if 'assigned_to' in issue:
        assigned_to_name = issue['assigned_to']['name']
    print('被分派者:', assigned_to_name)

def update_issue(issue_no):
    payload = {
        'issue': {
            'status_id': '5',  # 結案
            'notes': 'Hello world'
        }
    }
    res = requests.put(f'{url}{issue_no}.json', 
                       verify=False, 
                       headers={'X-Redmine-API-Key': api_key}, 
                       json=payload)
    if res.status_code != requests.codes.ok:
        print('失敗', res.status_code)
        return
    print('成功')
```

# 填工時

請求內容使用xml格式，資料需encode()後才代入data參數。

```python
import requests


requests.packages.urllib3.disable_warnings()  # 關閉InsecureRequestWarning
url = 'https://$redmine_url/time_entries.xml'
api_key = '$api_key'

def input_time_entry(issue_id, activity_id, hours, comments, date):
    payload = f'<time_entry issue_id="{issue_id}" ' \
              f'activity_id="{activity_id}" hours="{hours}" ' \
              f'comments="{comments}" spent_on="{date}" />'
    print(payload)
    res = requests.post(url,
                        verify=False,
                        headers={'X-Redmine-API-Key': api_key, 'Content-Type': 'application/xml'},
                        data=payload.encode())
    print(res.status_code)
```