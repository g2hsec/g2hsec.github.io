---
layout: single
title: (bWAPP)Host Header Attack (Reset Poisoning)
categories: bwapp
tag: [spoofing,mitm, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> Host Header Attack의 경우 웹 어플리케이션이 호스트 헤더를 신뢰하고 검증하지 않을 때 발생하며, 호스트 헤더는 클라이언트가 서버에 요청하는 도메인 이름을 나타낸다. 이로 인해, 사용자 데이터 유출, 세션 하이재킹, xss 등의 보안취약점이 발생할 수 있다.

![그림 1-1](/assets/image/bwapp/sensitive%20data%20exposure/Host%20Header%20Attack%20(Reset%20Poisoning)/image.png)
- 비밀번호 재설정 및 변경을 위해 이메일을 입력하게 되어 있다.

![그림 1-2](/assets/image/bwapp/sensitive%20data%20exposure/Host%20Header%20Attack%20(Reset%20Poisoning)/image-1.png)
- 희생자 pc에서 이메일 입력 후 전송될 때 패킷을 가로채
- host 헤더에 공격자의 웹 서버 주소를 기입할 경우
- 해당 secret 값이 공격자의 웹 서버로 넘어가게 된다.

```
해당 문제는 smtp 기능이 동작되지 않아 실습이 제한됨
```