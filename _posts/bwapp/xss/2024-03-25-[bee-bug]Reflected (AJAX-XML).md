---
layout: single
title: (bWAPP)XSS - Reflected (AJAX/XML)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(AJAX-XML)-archive/image.png)
- 영화 검색기능이 존재한다.
- 이 떄 데이터의 요청과 응답이 AJAX-XML 형식으로 전달되는 듯 하다.

```
HTTP/1.1 200 OK
Date: Sun, 31 Mar 2024 12:58:47 GMT
Server: Apache/2.2.8 (Ubuntu) DAV/2 mod_fastcgi/2.4.6 PHP/5.2.4-2ubuntu5 with Suhosin-Patch mod_ssl/2.2.8 OpenSSL/0.9.8g
X-Powered-By: PHP/5.2.4-2ubuntu5
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Content-Length: 118
Connection: close
Content-Type: text/xml; charset=utf-8

<?xml version="1.0" encoding="UTF-8" standalone="yes"?><response>test??? Sorry, we don't have that movie :(</response>
```

- 버프스위트를 통한 응답값을 확인해보았다.
- 요청은 AJAX형식으로 이루어지며, 응답값이 XML 형태로 반환된다.
- 또한 요청값은 GET 형식으로 전달된다.

```javascript
function process()
{
    // Proceeds only if the xmlHttp object isn't busy
    if(xmlHttp.readyState == 4 || xmlHttp.readyState == 0)
    {
        // Retrieves the movie title typed by the user on the form
        title = encodeURIComponent(document.getElementById("title").value);
        // Executes the 'xss_ajax_1-2.php' page from the server
        xmlHttp.open("GET", "xss_ajax_1-2.php?title=" + title, true);  
        // Defines the method to handle server responses
        xmlHttp.onreadystatechange = handleServerResponse;
        // Makes the server request
        xmlHttp.send(null);
    }
    else
        // If the connection is busy, try again after one second  
        setTimeout("process()", 1000);
}
```

- 해당 페이지의 공개된 소스코들 살펴보면, GET형식으로 데이터를 xss_ajax_1-2.php 파일로 전송하는 것을 확인할 수 있다.

```php
if(isset($_GET["title"]))
{

    // Creates the movie table
    $movies = array("CAPTAIN AMERICA", "IRON MAN", "SPIDER-MAN", "THE INCREDIBLE HULK", "THE WOLVERINE", "THOR", "X-MEN");

    // Retrieves the movie title
    $title = $_GET["title"];

    // Generates the XML output
    header("Content-Type: text/xml; charset=utf-8");

    // Generates the XML header
    echo "<?xml version=\"1.0\" encoding=\"UTF-8\" standalone=\"yes\"?>";

    // Creates the <response> element
    echo "<response>";

    // Generates the output depending on the movie title received from the client
    if(in_array(strtoupper($title), $movies))
        echo "Yes! We have that movie...";
    else if(trim($title) == "")
        echo "HINT: our master really loves Marvel movies :)";
    else
        echo xss($title) . "??? Sorry, we don't have that movie :(";

    // Closes the <response> element
    echo "</response>";

}
```

- 검증 코드를 보게되면 Content-Type 헤더가 text.xml 로 지정되어 있다.
- 이로 인해 javascript가 실행되지 않으며, xml형식으로 변환되기에 <> 또한 적용되지 않을 것이다.
- 이 때 HTML Entity Encoding 을 통해 우회할 수 있다.

```
&lt;img src='x'; onerror='alert(1)'&gt;
```

- 위와 같이 <> 혹은 ",' 와 같은 특수 문자들을 HTML Entity Encoding이 가능하다.
- 변조 후 입력값을 넣게되면

![그림 1-2](/assets/image/bwapp/xss/Reflected%20(AJAX-XML)-archive/image-1.png)

- 성공적으로 alert 창이 나타난다.

## Level - Medium & High

```php
function xss_check_3($data, $encoding = "UTF-8")
{
    // htmlspecialchars - converts special characters to HTML entities    
    // '&' (ampersand) becomes '&amp;' 
    // '"' (double quote) becomes '&quot;' when ENT_NOQUOTES is not set
    // "'" (single quote) becomes '&#039;' (or &apos;) only when ENT_QUOTES is set
    // '<' (less than) becomes '&lt;'
    // '>' (greater than) becomes '&gt;'  
    return htmlspecialchars($data, ENT_QUOTES, $encoding);
}
function xss_check_4($data)
{
    // addslashes - returns a string with backslashes before characters that need to be quoted in database queries etc.
    // These characters are single quote ('), double quote ("), backslash (\) and NUL (the NULL byte).
    // Do NOT use this for XSS or HTML validations!!!
    return addslashes($data);
}
```

- Medium 의 경우 addslashes를 통해 ' " null 문자들에 대한 필터링이 이루어지고 있다.
- 하지만 해당 레벨을 HTML Entity Encoding를 통해 ' 또한 바꿔주게 되면 우회가 가능하다.

```
&lt;img src=&apos;x&apos; onerror=&apos;alert(1)&apos;&gt;
```
![그림 1-3](/assets/image/bwapp/xss/Reflected%20(AJAX-XML)-archive/image-2.png)

- High 의 경우 htmlspecialchares 함수를 통해 주석에 나와있는 5가지 메타 문자에 대한 필터링이 이루어지고 있다.
- 이 경우 해당 문제에서 HTML Entity 값을 다시금 HTML 로 변환하는 과정을 거치므로
- 메타문자를 그대로 사용할시 htmlspecialchars 함수를 통해 HTML Entity Encoding가 이루어지고
- 이 때 다시금 HTML Entity 는 HTML로 변환되어 클라이언트에 반환되므로 스크립트가 실행 된다.
- 이러한 이유는 클라이언트 측에서 HTML Entity 인 &lt; 와 &gt; 같은 문자를 HTML Entity로 인식하지 못하여, &lt; &gt;를 다시 <> 와 같은 일반적인 HTML태그로 해석하는 과정에서 발생한다.
- Content-Type 헤더가 text:xml 의 경우 이와 같이 발생할 수 있다.

```
<img src='' onerror='alert(1)'>
```

![alt text](/assets/image/bwapp/xss/Reflected%20(AJAX-XML)-archive/image-3.png)