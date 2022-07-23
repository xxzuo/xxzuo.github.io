---
title: azkaban接口
author: xxzuo
date: 2022-07-23 23:45:44
tags:
categories:
- azkaban
---



## azkaban接口列表

| 接口                            | 说明     |
| ------------------------------- | -------- |
| [/manager?action=login](#login) | 登录接口 |


***


## 接口详情
* <span id = "login">登录接口</span>

	* 接口地址：/manager

	* 返回格式：Json

	* 请求方式：Post

	* 请求示例：https://localhost:8443/manager

	* 接口备注：This API helps authenticate a user and provides a session.id in response.

	* 请求参数说明：

        | 名称         | 类型   | 必填 | 说明     |
        | ------------ | ------ | ---- | -------- |
        | action=login | string | true | 登录参数 |
        | username     | string | true | 用户名   |
        | password     | string | true | 用户密码 |

	* 返回参数说明：

        | 名称       | 类型   | 说明     |
        | ---------- | ------ | -------- |
        | status     | int    | 状态码   |
        | session.id | string | 会话ID |
    
	* JSON返回示例：
	
	  ```json
	  {
	    "status" : "success",
	    "session.id" : "c001aba5-a90f-4daf-8f11-62330d034c0a"
	  }
	  ```
	
    
  



