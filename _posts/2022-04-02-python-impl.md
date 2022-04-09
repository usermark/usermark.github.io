---
layout: article
title: '[Python] 應用'
date: 2022-04-02 16:45
tags: Python
---
紀錄開發上常遇到的問題，避免重複踩坑。
<!--more-->
# Requests紀錄API請求和回應

參考<https://stackoverflow.com/questions/16337511/log-all-requests-from-the-python-requests-module>
```python
import logging
from http.client import HTTPConnection

def debug_requests_on():
    '''Switches on logging of the requests module.'''
    HTTPConnection.debuglevel = 1

    logging.basicConfig()
    logging.getLogger().setLevel(logging.DEBUG)
    requests_log = logging.getLogger("requests.packages.urllib3")
    requests_log.setLevel(logging.DEBUG)
    requests_log.propagate = True

def debug_requests_off():
    '''Switches off logging of the requests module, might be some side-effects'''
    HTTPConnection.debuglevel = 0

    root_logger = logging.getLogger()
    root_logger.setLevel(logging.WARNING)
    root_logger.handlers = []
    requests_log = logging.getLogger("requests.packages.urllib3")
    requests_log.setLevel(logging.WARNING)
    requests_log.propagate = False
```

# requests_html出現SSLCertVerificationError

第一次執行會有SSLCertVerificationError問題
```shell
[W:pyppeteer.chromium_downloader] start chromium download.
Download may take a few minutes.
Traceback (most recent call last):
  略...
ssl.SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1056)
```
修正如下，參考<https://github.com/miyakogi/pyppeteer/issues/219>
```shell
pip install -U "urllib3<1.25"
```