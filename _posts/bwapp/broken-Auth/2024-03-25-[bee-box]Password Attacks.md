---
layout: single
title: (bWAPP)Broken Auth. - Password Attacks
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/Broken-Auth/Password-attack-archive/image.png)
- 로그인 기능이 구현되어 있으며,
- 해당 시나리오는 부르트포스 및 사전대입공격을 사용하는 시나리오인듯 함

![그림 1-2](/assets/image/bwapp/Broken-Auth/Password-attack-archive/image-1.png)
- 로그인 실패시 위 그림1-2와 같은 문자가 출력됨

![그림 1-3](/assets/image/bwapp/Broken-Auth/Password-attack-archive/image-2.png)
- 로그인 시점에서 Intercept 를 통해 패킷을 잡은 후 Intruder로 보내 공격 페이로드를
- login, password 파라미터로 설정한다.

![그림 1-4](/assets/image/bwapp/Broken-Auth/Password-attack-archive/image-3.png)
![그림 1-5](/assets/image/bwapp/Broken-Auth/Password-attack-archive/image-4.png)
- 그 후 페이로드 1,2 를 각각 login, password 파라미터에 들어갈 사전파일을 지정한다.
- 현 실습에서는 단순한 몇개의 단어만 설정했다.

![그림 1-6](/assets/image/bwapp/Broken-Auth/Password-attack-archive/image-5.png)
- 그 후 매칭되는 문자열을 이전 로그인 실패시 출력되는 문자열로 설정 후 Attack 을 수행한다.
![그림 1-7](/assets/image/bwapp/Broken-Auth/Password-attack-archive/image-6.png)
- 실행 결과를 보게 되면
- paylaod1 → bee
- payload2 → bug
- 부분에서 매칭되는 문자열이 없다는 걸 확인할 수 있으며
- 이를 통해 계정정보를 탈취할 수 있다.