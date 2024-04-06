---
layout: single
title: (bWAPP)phpMyAdmin BBCode Tag XSS
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/bwapp/xss/phpMyAdmin%20BBCode%20Tag%20XSS-archive/image.png)
- phpmyadmin 내에 error.php 페이지의 BBcode에서 스크립트가 발생하는 취약점인 거 같다.
- CVE-2010-4480 으로 명명되어있다.
- 해당 취약점은 PhpMyAdmin 3.3.8.1 혹은 3.4.0-beta1 이전 버전에서 발생한다.

>BBcode 사용법은 [@url@page] 와 같은 형식으로 사용된다.

![그림 1-2](/assets/image/bwapp/xss/phpMyAdmin%20BBCode%20Tag%20XSS-archive/image-1.png)
- bWAPP의 최상위 루트에서 phpmyadmin 페이지가 존재한다.

- bwapp가 가동중인 시스템에서 /var/www/bWAPP 경로에 스크립트 자격증명을 진행할 html 파일을 하나 생성한다.

```
ex) phpmyadminxss.html

a<script> alert(1)</script>
```

![그림 1-3](/assets/image/bwapp/xss/phpMyAdmin%20BBCode%20Tag%20XSS-archive/image-2.png)
- 그 후 취약점이 존재하는 페이지인 error.php 에 접근한다
- \<ip>/phpmyadmin/error.php

```
error.php?type=test
```

![그림 1-4](/assets/image/bwapp/xss/phpMyAdmin%20BBCode%20Tag%20XSS-archive/image-3.png)
- type 라는 파라미터에 값을 입력하면 error 페이지 내에 입력값이 출력된다.

```
/phpmyadmin/error.php?type=test&error=a+b+c+[a@http://{bwapp_ip}/bWAPP/phpmyadminxss.html@]d+e[/a]
```

![그림 1-5](/assets/image/bwapp/xss/phpMyAdmin%20BBCode%20Tag%20XSS-archive/image-4.png)
- 위 URI 와 같이 BBcode를 통해 입력값을 전송하면 사진과 같이 하이링크가 하나 생성된다

![그림 1-6](/assets/image/bwapp/xss/phpMyAdmin%20BBCode%20Tag%20XSS-archive/image-5.png)
- 클릭할 경우 bwapp 서버에 생성해둔 phpmyadminxss.html 파일을 불러오며 스크립트가 실행된다.