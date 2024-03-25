---
layout: single
title: (bWAPP)XSS - Reflected (GET)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(GET)-archive/image.png)
- 사용자의 성, 이름 입력구간이 존재하며, 입력 값은 URI 를 통해 GET 형식으로 전송된다.

```shell
/bWAPP/xss_get.php?firstname=test&lastname=name&form=submit
```

![그림 1-2](/assets/image/bwapp/xss/Reflected%20(GET)-archive/image-1.png)

- 그 후 사용자의 입력 값이 서버 측에서 받아 동적 페이지를 구성 후 클라이언트 측에 반환 되는 걸 확인할 수 있다.
- 해당 XSS 취약점은 이를 통해 Reflected XSS 라는 것을 알 수 있다

![그림 1-3](/assets/image/bwapp/xss/Reflected%20(GET)-archive/image-2.png)
- 위 사진과 같이 사용자의 입력 값을 주는 구간에 스크립트를 삽입할 시 서버 측에서 해당 입력값을 받아 동적 페이지를 구성 후 클라이언트에게 반환할 때 브라우저 단에서는 해당 스크립트가 고스란히 실행되어  alert() 을 실행시킨

![그림 1-4](/assets/image/bwapp/xss/Reflected%20(GET)-archive/image-3.png)

- 해당 취약점의 예상 시나리오로 다양한 공격이 가능하겠지만, 일반적인 Reflected XSS 은 GET 형식으로 보내는 URI에 악성 스크립트를 담아 타 사용자에게 전송
- 해당 URI를 클릭한 타 사용자는 본인의 세션 값등의 정보를 공격자의 웹 서버로 전송될 수 있음.
- 이를 통해 다양한 공격이 가능하며, 피싱과 같은 공격도 가능하다.

## Level - Medium & High

![그림 1-5](/assets/image/bwapp/xss/Reflected%20(GET)-archive/image-4.png)
- Medium Level에서는 잘못된 보안 대책을 구현하여 문제가 된다.
- addslashes 함수를 사용한 것으로 확인되며,
- 해당 함수는 특수 문자 앞에 “/” 백슬래쉬를 추가하여 해당 특수 문자를 이스케이프 처리를 위한 함수이다.
- 하지만 이 때 포함되는 특수 문자는 따옴표(’), 쌍따옴표(”), 백슬래쉬(/). null문자 가 포함되어
- XSS 공격에 사용되는 <> 와 같은 특수문자는 필터링되지 않는다.

![그림 1-6](/assets/image/bwapp/xss/Reflected%20(GET)-archive/image-5.png)
High Level에서는 htmlsepcialchars 함수를 사용하여, 각각의 특수문자에 대한 HTML 엔터티 인코딩을 통한 필터링을 적용하여, XSS 에 대한 대응이 이루어지고 있다.