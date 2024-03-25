---
layout: single
title: (bWAPP)Session Mgmt. - Cookies (HTTPOnly)
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image.png)
- Cookies 버튼을 클릭시 아래 표에 쿠키 값이 출력 되며,
- here 버튼 클릭시 alert 창을 통해 쿠키 값이 출력 된다.

![그림 1-2](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-1.png)
![그림 1-3](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-2.png)
- 세션 값과 각각의 쿠키값을 보여준다.
- 해당 시나리오는 세션값을 탈취하는 시나리오로 예상되며,
- test계정을 생성 후 test 계정으로 실습을 진행했다.
- XSS 와 같은 취약점과 연계하여 특정 사용자의 세션을 탈취했다고 가정하였다.

![그림 1-4](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-3.png)
- bee(관리자) 계정의 세션값을 확인 하였다.

![그림 1-5](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-4.png)
![그림 1-6](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-5.png)
- 그 후 test 계정으로 다시 접속 후
- 해당 페이지로의 요청값에 전송되는 쿠키 값을 확인했다.

```
POST /bWAPP/smgmt_cookies_httponly.php HTTP/1.1
Host: 192.168.146.133
Cookie: security_level=0; PHPSESSID=e92b70d89df8851c9a5b4793e77eb53b; top_security=no
Content-Length: 12
Cache-Control: max-age=0
Sec-Ch-Ua: "Not_A Brand";v="8", "Chromium";v="120"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.6099.216 Safari/537.36
Origin: https://192.168.146.133
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://192.168.146.133/bWAPP/smgmt_cookies_httponly.php
Accept-Encoding: gzip, deflate, br
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Priority: u=0, i
Connection: close

form=cookies
```

- Cookie 헤더를 보면 PHPSESSID 값이 test 유저의 세션값으로 설정되어 있는 걸 볼 수 있다.
- 해당 값을 시나리오상 탈취한 bee(관리자) 계정의 세션 값으로 수정 후 요청을 전송했다.

> 해당 실습에서는 세션 관리로 인해 서로 다른 브라우저에서 진행하였다.

![그림 1-7](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-6.png)
- 위 사진 과 같이 test 계정에서 세션값 변조를 통해 Bee 계정으로 변경된 걸 알 수 있다.

> HTTP Only 란 무엇인가? - 사용자의 Client 에서 script 사용이 불가능 하도록 해주는 쿠키 헤더이며, 이는 XSS 와 같은 공격이 Java Script 를 통해 세션값 탈취를 하는 것을 막아주는 역할을 할 수 있다.

- 위 요청헤더에도 HTTP Only 쿠키 헤더가 없으며, 
![그림 1-8](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-7.png)
- Http Only 옵션이 비활성화 된 걸 알 수 있다.

> Http Only 외에도 XSS 와 같은 공격에서 세션 탈취를 방지하기 위해 Secure 헤더 및 세션이 유지되는 기간을 짧게 설정하는 방법들이 존재한다.

## Level - Medium & High

> Medium Level 에서는 Http Only 헤더를 활성화 시켜, Script를 통한 세션값 탈취가 불가능 하도록 되어 있다.

![그림 1-9](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-8.png)
- High Level 에서는 expires, maxAge 헤더를 통해 세션의 유효기간을 5분으로 설정하고 있다. 이를 통해 5분이 지난 후 세션은 종료되어, 재사용이 불가능하다.

![그림 1-10](/assets/image/bwapp/Broken-Auth/Cookies%20(HTTPOnly)-archive/image-9.png)
