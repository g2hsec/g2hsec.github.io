---
layout: single
title: (bWAPP)XSS - Reflected (POST)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(POST)-archive/image.png)
- First name과 Last name 파라미터로 넘어가는 사용자 입력값이 GET 형식이 아닌
- POST 형식으로 넘어간다.

```shell
firstname=test&lastname=test&form=submit
```

- burp suite를 통해 요청 패킷을 확인해보면 파라미터가 넘어가는 것을 확인할 수 있다.

```shell
firstname=<script>alert(1)</script>&lastname=test&form=submit
```

![그림 1-2](/assets/image/bwapp/xss/Reflected%20(POST)-archive/image-1.png)
- 위와 같이 POST형식으로 넘어가는 파라미터에 스크립트를 삽입 하여, 그림1-2와 같이 alert창을 띄워 XSS 취약점 존재 유/무 확인이 가능하다.
- 확인 결과, 스크립트가 고스란히 전달되어 서버측에서 동적 웹 페이지 구성 후 클라이언트에게 반환되어 렌더링 될 때 악성 스크립트가 실행된다.

## Level - Medium & High

![그림 1-3](/assets/image/bwapp/xss/Reflected%20(POST)-archive/image-2.png)
- Medium Level에서는 잘못된 보안 대책을 구현하여 문제가 된다.
- addslashes 함수를 사용한 것으로 확인되며,
- 해당 함수는 특수 문자 앞에 “/” 백슬래쉬를 추가하여 해당 특수 문자를 이스케이프 처리를 위한 함수이다.
- 하지만 이 때 포함되는 특수 문자는 따옴표(’), 쌍따옴표(”), 백슬래쉬(/). null문자 가 포함되어
- XSS 공격에 사용되는 <> 와 같은 특수문자는 필터링되지 않는다.

![그림 1-4](/assets/image/bwapp/xss/Reflected%20(POST)-archive/image-3.png)
- High Level에서는 htmlsepcialchars 함수를 사용하여, 각각의 특수문자에 대한 HTML 엔터티 인코딩을 통한 필터링을 적용하여, XSS 에 대한 대응이 이루어지고 있다.

## Level - Medium & High

![그림 1-3](/assets/image/bwapp/xss/Reflected%20(POST)-archive/image-2.png)
- Medium Level에서는 잘못된 보안 대책을 구현하여 문제가 된다.
- addslashes 함수를 사용한 것으로 확인되며,
- 해당 함수는 특수 문자 앞에 “/” 백슬래쉬를 추가하여 해당 특수 문자를 이스케이프 처리를 위한 함수이다.
- 하지만 이 때 포함되는 특수 문자는 따옴표(’), 쌍따옴표(”), 백슬래쉬(/). null문자 가 포함되어
- XSS 공격에 사용되는 <> 와 같은 특수문자는 필터링되지 않는다.

![그림 1-4](/assets/image/bwapp/xss/Reflected%20(POST)-archive/image-3.png)
- High Level에서는 htmlsepcialchars 함수를 사용하여, 각각의 특수문자에 대한 HTML 엔터티 인코딩을 통한 필터링을 적용하여, XSS 에 대한 대응이 이루어지고 있다.