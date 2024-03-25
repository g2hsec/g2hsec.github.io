---
layout: single
title: (bWAPP)Broken Auth. - Insecure Login Forms
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/Broken-Auth//insecure-login-forms-archive/image.png)
- 로그인 기능이 구현되어 있음
- 해당 시나리오의 경우 DB가 구성되어 있지 않거나, 웹 페이지 소스상에 로그인 정보가 고스란히 하드코딩 되어있는 경우를 나타냄
- 개발자의 실수를 통해 일어날 수 있음

![그림 1-2](/assets/image/bwapp/Broken-Auth//insecure-login-forms-archive/image-1.png)
- 해당 부분을 마우스를 통해 드래그하거나 소스코드를 통해 확인해보면 계정정보가 노출되어 있다.
- 소스코드에서는 화면에 출력되지 않게 컬러를 화이트로 설정해놓은 것을 알 수 있다.
> 해당 취약점의 경우 DB 사용을 통해 사용자의 정보를 보관 및 관리해야하며, 소스코드상에 하드코딩 하지 않아야함.