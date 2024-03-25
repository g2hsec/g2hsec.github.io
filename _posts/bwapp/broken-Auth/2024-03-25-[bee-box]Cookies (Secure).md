---
layout: single
title: (bWAPP)Session Mgmt. - Cookies (Secure)
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/Broken-Auth/Cookies%20(Secure)-archive/image.png)
- 이전 문제의 Http Only 와 문제 자체는 동일하지만, Http Only 헤더가 아닌
- Secure 헤더에 대한 학습을 위한 문제이다.

> Secure 헤더란 SSL/TLS 을 사용한 HTTPS 프로토콜을 통해 전송되는 응답에만 쿠키를 포함하도록 브라우저에 지시하는 헤더이다. 이를 통해 XSS 를 통한 세션 하이재킹, 중간자 공격, CSRF 등의 공격에 대응이 가능하다.

![그림 1-2](/assets/image/bwapp/Broken-Auth/Cookies%20(Secure)-archive/image-1.png)
![그림 1-3](/assets/image/bwapp/Broken-Auth/Cookies%20(Secure)-archive/image-2.png)

- 실습은 이전과 동일하게 서로 다른 브라우저에서 진행하였다.
- bee 계정의 세션값을 탈취 후 test 계정에서 세션을 변조하여 bee 계정으로 변경된 걸 확인할 수 있다.


![그림 1-4](/assets/image/bwapp/Broken-Auth/Cookies%20(Secure)-archive/image-3.png)
- Low Level 에서는 secure 헤더가 적용되어 있지 않기에 가능하다.


## Level - Medium & High

- Medium Level 에서는 secure 헤더가 적용되어 있다.
- High Level 에서는 이전과 동일하게 세션의 유효기간이 5분으로 설정되어 있다.

![그림 1-5](/assets/image/bwapp/Broken-Auth/Cookies%20(Secure)-archive/image-4.png)
- 일반적으로 Secure 헤더가 적용되어 있지 않아 HTTP프로토콜을 통해 데이터 전송을 하게되면,
- 그림 1-3 과 같이 와이어샤크를 통해 패킷을 캡처해보면 쿠키값이 고스란히 노출되고 있음을 알 수 있다.
- 이 때 보안을 위해 Secure 헤더를 사용하여 SSL/TLS 를 통한 HTTPS 통신을 하게되었을 경우의 패킷을 확인해보았다.
![그림 1-6](/assets/image/bwapp/Broken-Auth/Cookies%20(Secure)-archive/image-5.png)
- TLS 프로토콜을 통한 데이터 전송을 수행하며, 암호화를 통해 내용확인이 불가능 하다.