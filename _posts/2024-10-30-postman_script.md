---
layout: post
title: "Postman调用登录接口自动设置token"
date: 2024-10-30
tags: [测试]
---
# Postman调用登录接口自动设置token

## 将登录接口报文中token设置到Collection变量中

登录接口 Scripts Post-response设置

auth_token为变量名

response.data.token为json中token字段,根据实际报文结构修改
![alt text](/assets/images/postman_script_post_response.png)
```js
const response = pm.response.json();
pm.collectionVariables.set("auth_token", response.data.token);
```

## Collection Authorization读取使用auth_token变量

根据实际情况修改 



Auth Type: API Key

Key:token

Value:{{auth_token}}

Add to:Header