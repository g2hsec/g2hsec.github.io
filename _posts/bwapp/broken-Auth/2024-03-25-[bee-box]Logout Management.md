---
layout: single
title: (bWAPP)Broken Auth. - Logout Management
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/Broken-Auth/logout-management-archive/image.png)
- 간단한 로그아웃 기능이 구현되어 있음
- 해당 시나리오의 경우 “뒤로가기” 기능을 통해 이전 사용자의 세션을 가지고 다시금 로그인 상태로 돌아갈 수 있음.

![그림 1-2](/assets/image/bwapp/Broken-Auth/logout-management-archive/image-1.png)
![그림 1-3](/assets/image/bwapp/Broken-Auth/logout-management-archive/image-2.png)
- 성공적으로 로그 아웃되어 BWAPP 초기 화면으로 돌아왔다.

![그림 1-4](/assets/image/bwapp/Broken-Auth/logout-management-archive/image-3.png)
- “뒤로가기” 를 통해 돌아갈 경우 이전의 세션을 가지고 다시 로그인 상태로 돌아간 것을 확인할 수 있음

> 해당 취약점의 경우 로그아웃시 사용자의 세션을 완전하게 삭제조치 해야함.