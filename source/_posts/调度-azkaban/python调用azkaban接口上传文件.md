---
title: python调用azkaban接口上传文件
author: xxzuo
tags:
  - azkaban
categories:
  - 调度-azkaban
date: 2023-07-13 23:55:11
---


python调用

```python
import os.path  
  
import requests  
  
def login_azkaban():  
    headers = {  
        "Content-Type": "application/x-www-form-urlencoded"  
    }  
    data = {  
        'action': 'login',  
        'username': 'azkaban',  
        'password': 'azkaban',  
    }  
    response = requests.post('http://192.168.x.x:8081/manager', data=data, headers=headers)  
    res_data = response.json()  
    if "error" in res_data:  
        return ""  
    return res_data["session.id"]  
  
  
def upload_project_zip(session_id, project_name, zip_file):  
    files = {  
        'file': (  
            os.path.basename(zip_file), open(zip_file, 'rb'), 'application/zip')  
        ,  
        'ajax': (None, 'upload', 'application/x-www-form-urlencoded'),  
        'project': (None, project_name, 'type/plain'),  
        'session.id': (None, session_id, 'application/x-www-form-urlencoded')  
    }  
    response = requests.post(  
        'http://192.168.x.x:8081/manager', files=files)  
    return response.json()
```
