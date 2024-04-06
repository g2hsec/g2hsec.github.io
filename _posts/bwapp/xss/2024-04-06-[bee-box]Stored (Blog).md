---
layout: single
title: (bWAPP)Stored (Blog)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](image.png)
- 글쓰기 기능이 존재한다.
- 대표적인 Stored XSS 취약점이 발견되는 유형인 게시글 유형이다.

![그림 1-2](image-1.png)
- 글을 쓰면 사용자의 입력값은 서버 데이터베이스에 저장되고
- 게시글을 볼 떄 데이터베이스에 저장되어 있는 데이터를 가져와 페이지 상에 출력하게 된다.
- 이 때 사용자의 입력값이 악성 스크립트가 주입될 경우
- 악성 스크립트는 고스란히 서버 데이터베이스 측에 저장되고
- 게시글을 불러올 때 마다 지속적으로 스크립트가 실행되게 된다.

![그림 1-3](image-2.png)
![그림 1-4](image-4.png)
![그림 1-5](image-3.png)
- 악성 스크립트를 주입하게 되면 위 사진과 같이 게시글을 불러오거나 새로고침하여 해당 게시글을 볼 때 마다 스크립트가 실행되게 된다.