---
layout: article
title: '[Python] 爬蟲練習-登入'
date: 2022-03-30 07:22
tags: Python
---
以[財政部電子發票整合服務平台](https://www.einvoice.nat.gov.tw/)的登入做練習。利用Chrome的開發人員工具分析後，用python來實現登入行為。
<!--more-->
{% note warning %}
僅供學習，切勿用於非法用途！
{% endnote %}

# 分析

開啟Chrome的開發人員工具後，要記得將Preserve log選項打開，才不會因為網頁切換造成log被清掉。

![](/assets/invoice_login.png)
點擊登入按鈕時，會重新產生驗證碼randNum，送出後先跑Ajax，當回應num為0時表示驗證碼正確，才跑Login。

![](/assets/invoice_rand.png)
此處去找出被呼叫js程式碼為
```js
function changePicIndex(){
    if($("#randPicIndex").length > 0 && $("#evn_geobox").css("display") == "table"){
        document.getElementById("randPicIndex").src="/home/randNum?id="+Math.random(); 
    }
}
```

![](/assets/invoice_ajax.png)
此處去找出被呼叫js程式碼為
```js
function checkPicAndSubmit(){
    var url = '/home/Ajax';
    $("#recaptchaValue").val($('#checkPicIndex').val());
        var params = {
            checkString:$('#checkPicIndex').val()
        };
        return jQuery.post(url, params, function callbackFun(data){
            if(data.num == '1'){
                changePicIndex();
                alert("圖形驗證碼輸入錯誤\n");
            } else {
                checkStatus = true
                doLogin();
            }
        }, 'json');
}
```

![](/assets/invoice_cookie.png)
這裡有些專有名詞可以先去了解下：
1. JSESSIONID: 是從tomcat來的，透過sessionId來區分不同用戶。
2. HttpOnly, Secure, SamSite: <https://tech-blog.cymetrics.io/posts/jo/zerobased-secure-samesite-httponly/>
3. Referrer Policy: <https://www.maxlist.xyz/2020/08/03/chrome-85-referer-policy/>
4. xhr(XMLHttpRequest): 可以不用刷新整個頁面，也能更新網頁中的部分資料。

# 實作

由於有JSESSIONID辨識用戶，為了確保是同一個連線，需使用requests.session()。

完整程式碼
```python
import json
import random

import requests
from bs4 import BeautifulSoup

user_agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.83 Safari/537.36'

def login():
    session = requests.session()
    
    # 產生驗證碼
    url = "https://einvoice.nat.gov.tw/home/randNum?id=" + str(random.random())
    headers = {
        'Host': "www.einvoice.nat.gov.tw",
        'Referer': "https://einvoice.nat.gov.tw/index",
    }
    res = session.get(url, stream=True, headers=headers)
    print('產生驗證碼', url, res.status_code)
    if res.status_code != requests.codes.ok:
        return

    # 下載圖片
    with open('rand.jpg', 'wb') as file:
        for chunk in res:
            file.write(chunk)
    
    # 手動輸入
    phone = input('輸入手機號碼:')
    pw = input('輸入密碼:')
    captcha = input('輸入驗證碼:')
    
    # 跑Ajax檢查驗證碼
    url = 'https://www.einvoice.nat.gov.tw/home/Ajax'
    payload = {
        'checkString': captcha
    }
    headers = {
        'User-Agent': user_agent
    }
    res = session.post(url, data=payload, headers=headers)
    print('檢查驗證碼', url, res.status_code)
    if res.status_code != requests.codes.ok:
        return
    num = json.loads(res.text)['num']
    print('num=' + str(num))
    if num == 1:
        print('圖形驗證碼輸入錯誤')
        return
    
    # 登入
    url = 'https://www.einvoice.nat.gov.tw/Login'
    payload = {
        'userType': 'N',
        'loginType': 'U',
        'loginWay': 'W',
        'serviceType': 'I',
        'serial': '',
        'pincode': pw,
        'signatur': '',
        'ban': '',
        'userID': '',
        'password': pw,
        'pid': '',
        'orgType': '',
        'bindata': '',
        'typeCheck': '1',
        'mobile': phone,
        'urlMethod': '',
        'loginStatus': 'U',
        'userTypeStatus': 'N',
        'recaptchaValue': captcha,
        'g-recaptcha-response': '',
        'rclink': ''
    }
    headers = {
        'User-Agent': user_agent,
        'Host': "www.einvoice.nat.gov.tw",
        'Referer': "https://einvoice.nat.gov.tw/index",
        'Origin': "https://www.einvoice.nat.gov.tw"
    }
    res = session.post(url, data=payload, headers=headers)
    print('登入', url, res.status_code)
    if res.status_code != requests.codes.ok:
        return
    
    # 儲存網頁
    with open('BTC508W.html', 'wb') as file:
        file.write(res.content)
    soup = BeautifulSoup(res.text, 'html.parser')
    print('title:', soup.title.string)
    startIndex = res.text.index('userInternal.setUser(') + 21
    endIndex = res.text.index(');', startIndex)
    data = res.text[startIndex:endIndex]
    # print(data)
    print('手機條碼:', json.loads(data)['carrierId2'])

if __name__ == "__main__":
    login()
```