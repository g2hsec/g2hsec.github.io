---
layout: single
title: (bWAPP)Insecure WebDAV Configuration
categories: bwapp
tag: [Local Privilege Escalation, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> 해당 시나리오는 CVE-2009-2692로 명명된 취약점을 이용한 로컬 권한 상승 취약점이다. 이는 net/socket.c의 sock_sendpage()에서 발생하는 NULL Pointer Derference취약점이다. 

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Local%20Privilege%20Escalation%20(sendpage)/image.png)
- 우선 로컬 권한 상승을 위해서는 해당 서버로의 침투가 우선시 되어야 한다.
- 이전 시나리오의 webdav 경로에 reverse connetction이 가능한 스크립트 파일을 업로드하여 reverse connetcion을 시도한다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Local%20Privilege%20Escalation%20(sendpage)/image-1.png)
- 성공적으로 서버내 웹서버 사용자로 내부침투에 성공했다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Local%20Privilege%20Escalation%20(sendpage)/image-2.png)
- cve-2009-2692를 이용한 공격 스크립트들이 존재한다.
- 이를 내부 서버로 옮겨 실행해볼 수 있을 듯 하다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Local%20Privilege%20Escalation%20(sendpage)/image-3.png)
- 공격자 서버에서 공격 스크립트를 준비 후
- 웹 서버를 구동시킨다.

