---
layout: single
title: (bWAPP)Man-in-the-Middle Attack (SMTP)
categories: bwapp
tag: [mitm, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> SMTP는 전자 메일을 사용하는데 사용되는 프로토콜이다. 이 또한 에이전트와 클라이언트 사이의 통신을 담당 하며, 메시지를 수락하고 전달하는데 사용된다. 이러한 SMTP가 중간자 공격에 노출될 경우 도청, 메시지 조작, 주소 위조등의 보안 위협으로 이어질 수 있다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Man-in-the-Middle%20Attack%20(SMTP)/image-1.png)
- 비밀번호를 잊어버렸을 경우 시크릿 번호를 메일로 보내주는 기능인 듯 하다.
- 이 경우 SMTP프로토콜을 사용하는 듯 하다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Man-in-the-Middle%20Attack%20(SMTP)/image.png)
- ettercap 를 사용하여 중간자 공격을 실습 해보았다.
- 우선 Host scan 을 통해 확인했을 때 동일 네트워크 대역대에
- bWAPP 서버와 희상자 PC인 우분투(X.X.X.138)이 잡혀있다.
- 두 호스트를 각각 타겟 1과 2로 지정하였다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Man-in-the-Middle%20Attack%20(SMTP)/image-2.png)
- 타겟 선택 후 ARP 스푸핑을 진행 하였다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Man-in-the-Middle%20Attack%20(SMTP)/image-3.png)
- 희생자 PC인 우분투에서 Secret 값을 전송 시켜 보았다.

![그림 1-5](/assets/image/bwapp/Security%20Misconfiguration/Man-in-the-Middle%20Attack%20(SMTP)/image-4.png)
- 와이어 샤크로 패킷을 확인해보면 SMTP로 전송된 데이터들을 확인할 수 있다.