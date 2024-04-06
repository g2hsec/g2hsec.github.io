---
layout: single
title: (bWAPP)Stored (Blog)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/bwapp/xss/Stored%20(Change%20Secret)-archive/image.png)
- 시크릿 값(패스워드 힌드)를 변경하는 로직이 구현되어 있는 듯 하다.

![그림 1-2](/assets/image/bwapp/xss/Stored%20(Change%20Secret)-archive/image-1.png)
- 입력값을 주고 변경을 하더라도 입력값이 페이지 내 혹은 응답값 내에 출력되지는 않는다.

![그림 1-3](/assets/image/bwapp/xss/Stored%20(Change%20Secret)-archive/image-2.png)
- 위와 같이 로그인 실패시 패스워드 힌트에 대한 secret 값에 test를 주었다.

![그림 1-4](/assets/image/bwapp/xss/Stored%20(Change%20Secret)-archive/image-3.png)
- 그 후 로그인 기능이 구현되어 있는 문제 페이지에서 로그인시
- 로그인 성공과 함꼐 변경한 secret 값이 노출되고 있다.
- 이 때 secret 값에 악성 스크립트를 주입하게 되면 해당 악성 스크립트는 서버 데이터베이스에 저장되고 위 사진과 같이 secret 값을 불러와 페이지상에 출력할 때 악성 스크립트가 실행되게 된다.


![그림 1-5](/assets/image/bwapp/xss/Stored%20(Change%20Secret)-archive/image-4.png)
- 위와 같이 입력 후

![그림 1-6](/assets/image/bwapp/xss/Stored%20(Change%20Secret)-archive/image-6.png)
![그림 1-7](/assets/image/bwapp/xss/Stored%20(Change%20Secret)-archive/image-5.png)
- 로그인 폼에서 로그인시 secret 출력 구간에서 스크립트가 발생하는 것을 볼 수 있다.