---
layout: single
title: (bWAPP)Session Mgmt. - Strong Sessions
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/Broken-Auth/Strong%20Sessions-archive/image.png)
- 이전 시나리오의 내용들을 합친 시나리오라고 볼 수 있다.
- 각각의 Level 마다 점점 강한 세션관리를 통한 세션 처리가 이루어지는 듯 하다.

![그림 1-2](/assets/image/bwapp/Broken-Auth/Strong%20Sessions-archive/image-1.png)
- Low Level 의 경우 이전과 동일하게 탈취한 세션을 통한 변조 후 해당 페이지 접근시정상적으로 접근하게 된다.

## Level - Medium & High

- Medium Level 의 경우 Http Only 가 설정되어 있으며,
- Hihg Level 의 경우SSL/TLS 를 통한 HTTPS프로토콜로만 통신이 가능하도록 Secure 헤더가 설정되어 있다.