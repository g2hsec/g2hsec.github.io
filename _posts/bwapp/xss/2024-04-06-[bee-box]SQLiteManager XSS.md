---
layout: single
title: (bWAPP)SQLiteManager XSS
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss, CVE-2012-5105]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/bwapp/xss/SQLiteManager%20XSS-archive/image.png)

- SQLite의 관리자 페이지인 main.php 및 index.php 취약점을 악용한 xss취약점이다.
- CVE-2012-5105 으로 명명되어있다.
- 해당 취약점은 SQLLiteManager 1.2.4 에서 발생한다.

![그림 1-2](/assets/image/bwapp/xss/SQLiteManager%20XSS-archive/image-1.png)
- SQLiteManager version 1.2.4의 페이지가 존재하고 있다.

![그림 1-3](/assets/image/bwapp/xss/SQLiteManager%20XSS-archive/image-2.png)
- 좌측 메뉴중 임의의 일부 메뉴를 클릭하면 +Trigger 바가 보이게 된다.

![그림 1-4](/assets/image/bwapp/xss/SQLiteManager%20XSS-archive/image-3.png)
- 위 사진과 같이 Trigger을 작성하는 부분이 나오며
- test를 통해 출력포지션을 파악해본다.

![그림 1-5](/assets/image/bwapp/xss/SQLiteManager%20XSS-archive/image-4.png)
- 모두 출력되는 것을 볼 수 있다.

![그림 1-6](/assets/image/bwapp/xss/SQLiteManager%20XSS-archive/image-5.png)
- 위와 같이 두 부분에
- 악성 스크립트를 삽입한다.

![그림 1-7](/assets/image/bwapp/xss/SQLiteManager%20XSS-archive/image-7.png)
![그림 1-8](/assets/image/bwapp/xss/SQLiteManager%20XSS-archive/image-6.png)
- 작성된 트리거를 보면 출력구간에서 악성 스크립트가 실행되는 것을 알 수 있다.