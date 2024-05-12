---
layout: single
title: (bWAPP)Robots File
categories: bwapp
tag: [mitm, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> robots.txt 파일은 웹 크롤러에게 해당 웹 사이트의 클롤링 규칙을 알려주는 파일이다. 하지만 이러한 robtos.txt 파일이 노출될 경우 클롤링을 허용 및 거부하는 페이지가 노출되므로 다양한 경로가 노출될 수 있다. 즉, 악의적인 공격자에게 공격 표면을 제공하는 샘이다. 

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Robots%20File/image.png)
- bWAPP 서버에서 사용되는 robots.txt 파일인 듯 하다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Robots%20File/image-1.png)
- 실제 bWAPP 서버의 robots.txt 가 경로상에서 노출되고 있다.
- 해당 정보로 악의적인 사용자는 admin, documents, imoages, passwors 라는 경로가 존재하는 걸 확인했으며, 다양한 우회 공격을 통해 해당 공격 표면에 대한 공격이 가능 할 수 있다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Robots%20File/image-2.png)
- robots.txt 에 노출되어 있는 admin 페이지에 접근이 가능하며,
- 관리자 정보가 고스란히 노출되고 있다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Robots%20File/image-3.png)
![그림 1-5](/assets/image/bwapp/Security%20Misconfiguration/Robots%20File/image-4.png)
- passwords 경로로도 접근이 가능 했으며,
- 사용자들의 계정정보를 획득할 수 있었다.