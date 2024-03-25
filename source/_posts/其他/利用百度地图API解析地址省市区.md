---
title: 利用百度地图API解析地址省市区
author: xxzuo
categories:
  - 其他
date: 2024-02-03 23:17:35
tags:
---

对于一些详细地址，我们需要知道 这个地址它的标准省市区是什么。
比如 对于 `江苏省南京市玄武区鸡鸣寺路1号`  这个地址，要得到它的标准省市区 也就是
`江苏省-南京市-玄武区`

这里我们会用到百度地图的 两个 API 

- [地理编码 ](https://lbsyun.baidu.com/faq/api?title=webapi/guide/webservice-geocoding-base)
- [全球逆地理编码](https://lbsyun.baidu.com/faq/api?title=webapi/guide/webservice-geocoding-abroad)


1. 先使用 [地理编码 API](https://lbsyun.baidu.com/faq/api?title=webapi/guide/webservice-geocoding-base) 将 详细地址，转化为 地理坐标(经纬度)
2. 再利用 [全球逆地理编码](https://lbsyun.baidu.com/faq/api?title=webapi/guide/webservice-geocoding-abroad)  将 地理坐标(经纬度) 转化为 对应的位置信息 得到所在的行政区划

简单代码样例如下

```python
def get_location(ak, address):  
    geocoding_url = "https://api.map.baidu.com/geocoding/v3?location=showLocation"  
    params = {  
        "address": address,  
        "output": "json",  
        "ak": ak,  
        "extension_analys_level": 1,  
        "location": "showLocation"  
  
    }  

    try:  
        reverse_geocoding_url = "http://api.map.baidu.com/reverse_geocoding/v3/"  
        res = requests.post(url=geocoding_url, params=params)  
        if res:  
            res_json = res.json()  
            print(res_json)  
            if res_json['status'] == 0:  
                lng = res_json['result']['location']['lng']  
                lat = res_json['result']['location']['lat']  
                params2 = {  
                    "ak": ak,  
                    "output": "json",  
                    "coordtype": "bd09ll",  
                    "extensions_poi": "1",  
                    "location": str(lat) + "," + str(lng),  
                }  
  
                res2 = requests.post(url=reverse_geocoding_url, params=params2)  
                if res2:  
                    res_json2 = res2.json()  
                    print(res_json2)  
                    if res_json2['status'] == 0:  
                        return res2.json()['result']['addressComponent']  
    except Exception as e:  
        print(e)  
  
    return {"province": "", "city": "", "district": ""}
```

这里 `ak` 是 用户申请注册的key， `address` 是要解析的详细地址