---
title: 企业微信接口相关
categories:
  - 微信接口
date: 2023-10-23 10:01:08
tags:
  - wechat
author: xxzuo
---
## 1. 获取 access_token
### 1.1 术语
#### 1.1.1 secret
secret是企业应用里面用于保障数据安全的“钥匙”，每一个应用都有一个独立的访问密钥，为了保证数据的安全，secret务必不能泄漏。  
目前secret有：

- 自建应用secret。在管理后台->“应用与小程序”->“应用”->“自建”，点进某个应用，即可看到。
- 基础应用secret。某些基础应用（如“审批”“打卡”应用），支持通过API进行操作。在管理后台->“应用与小程序”->“应用->”“基础”，点进某个应用，点开“API”小按钮，即可看到。
- 通讯录管理secret。在“管理工具”-“通讯录同步”里面查看（需开启“API接口同步”）；
- 外部联系人管理secret。在“客户联系”栏，点开“API”小按钮，即可看到。

#### 1.1.2corpid
每个企业都拥有唯一的corpid，获取此信息可在管理后台“我的企业”－“企业信息”下查看“企业ID”（需要有管理员权限）

#### 1.1.3 userid
每个成员都有唯一的userid，即所谓“帐号”。在管理后台->“通讯录”->点进某个成员的详情页，可以看到。

#### 1.1.4 部门id
每个部门都有唯一的id，在管理后台->“通讯录”->“组织架构”->点击某个部门右边的小圆点可以看到

#### 1.1.5 tagid
每个标签都有唯一的标签id，在管理后台->“通讯录”->“标签”，选中某个标签，在右上角会有“标签详情”按钮，点击即可看到

#### 1.1.6 external_userid
企业外部联系人的id，可能是微信用户，也可能是企业微信用户。需要开发者(尤其是第三方开发者)注意的是，对于同一个外部联系人，不同调用方（企业/第三方服务商）获取到的external_userid是不同的。

#### 1.1.7 agentid
每个应用都有唯一的agentid。在管理后台->“应用与小程序”->“应用”，点进某个应用，即可看到agentid。


### 1.2 接口

获取access_token是调用企业微信API接口的第一步，相当于创建了一个登录凭证，其它的业务API接口，都需要依赖于access_token来鉴权调用者身份。  
因此开发者，在使用业务接口前，要明确access_token的颁发来源，使用正确的access_token。


>
> **请求方式：** GET（**HTTPS**）  
> **请求地址：** [https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=ID&corpsecret=SECRET](https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=ID&corpsecret=SECRET)  
> 
> 提示:  
> 此处标注大写的单词 ID 和 SECRET，为需要替换的变量，根据实际获取值更新。其它接口也采用相同的标注，不再说明。
 
  
**请求参数说明：**

|参数|必须|说明|
|---|---|---|
|corpid|是|企业ID|
|corpsecret|是|应用的凭证密钥，**注意应用需要是启用状态**|


**权限说明：**  
每个应用有独立的secret，获取到的access_token只能本应用使用，所以每个应用的access_token应该分开来获取

**返回结果：**
```json
{
   "errcode": 0,
   "errmsg": "ok",
   "access_token": "accesstoken000001",
   "expires_in": 7200
}
```

**响应参数说明：**

|参数|说明|
|---|---|
|errcode|出错返回码，为0表示成功，非0表示调用失败|
|errmsg|返回码提示语|
|access_token|获取到的凭证，最长为512字节|
|expires_in|凭证的有效时间（秒）|

**注意事项：**  
开发者需要缓存access_token，用于后续接口的调用（注意：不能频繁调用gettoken接口，否则会受到频率拦截）。当access_token失效或过期时，需要重新获取。

access_token的有效期通过返回的expires_in来传达，正常情况下为7200秒（2小时），有效期内重复获取返回相同结果，过期后获取会返回新的access_token。  
由于企业微信每个应用的access_token是彼此独立的，所以进行缓存时需要区分应用来进行存储。  
access_token至少保留512字节的存储空间。  
企业微信可能会出于运营需要，提前使access_token失效，开发者应实现access_token失效时重新获取的逻辑。


参考代码
```python
import requests
if __name__ == '__main__':
    res = requests.get(
        'https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=xxx=xxxxx')
    access = res.json()['access_token']
    print(access)
```

