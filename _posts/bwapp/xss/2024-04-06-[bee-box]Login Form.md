---
layout: single
title: (bWAPP)XSS - Reflected (Login Form)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(Login%20Form)-archive/image.png)
- 로그인 기능이 구현되어 있는 페이지이다.
- superhero 라는 유로 자격증명을 하라는 거 같지만, superhero 라는 유저는 존재하지 않는다.


![그림 1-2](/assets/image/bwapp/xss/Reflected%20(Login%20Form)-archive/image-1.png)
- ' 입력시 DBMS Error가 발생하게 된다.
- 해당 시나리오에서 sqli 취약점도 함께 존재한다.

![그림 1-3](/assets/image/bwapp/xss/Reflected%20(Login%20Form)-archive/image-2.png)
- '(싱글쿼터)와 함께 입력값을 주면 DBMS 가 출력될 떄 입력값도 함께 출력된다.

![그림 1-4](/assets/image/bwapp/xss/Reflected%20(Login%20Form)-archive/image-3.png)
- 스크립트 문을 작성해서 입력하면 위와 같이 출력된다.
- 이는 주석으로 sql 문을 끝내지 않아 발생하게 되는 오류이다.
- id 입력 부분의 SQL 쿼리 부분에서 임의적으로 종료 구문을 넣고 스크립트 실행이 가능하다.

```
'; <script>alert(1)</script>
```

![그림 1-5](/assets/image/bwapp/xss/Reflected%20(Login%20Form)-archive/image-4.png)
- alert가 실행되는 걸 볼 수 있다.