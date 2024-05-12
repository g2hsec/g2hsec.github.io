---
layout: single
title: (bWAPP)Insecure WebDAV Configuration
categories: bwapp
tag: [Local Privilege Escalation, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> 해당 취약점은 HTTP통신을 하게되면 평문으로 데이터를 주고 받게 되며, 이럴 경우 데이터 노출 및 ARP 스푸핑을 통해 스니핑 위험이 존재한다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Man-in-the-Middle%20Attack%20(HTTP)/image.png)
- HTTP를 통한 통신에서 로그인 기능이 구현되어 있다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Man-in-the-Middle%20Attack%20(HTTP)/image-1.png)
- 해당 실습을 위해 pc1(우분투) 에서 희생자pc bwapp 서버로의 중간 패킷을 가로채보기위해
- pc(우분투) 로의 arp 스푸핑을 진행했다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Man-in-the-Middle%20Attack%20(HTTP)/image-2.png)
- 우분투pc에서 bWAPP 접속후의 활동들이 칼리 리눅스의 와이어 샤크에
- 평문으로 고스란히 노출되고 있다.
- 이를통해 데이터 유출이 가능하다