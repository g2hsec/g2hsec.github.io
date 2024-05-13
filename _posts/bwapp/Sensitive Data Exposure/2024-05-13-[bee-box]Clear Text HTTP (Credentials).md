---
layout: single
title: (bWAPP)Clear Text HTTP (Credentials)
categories: bwapp
tag: [spoofing,mitm, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> 해당 시나리오의 취약점은 HTTP 통신을 하는 과정에서 암호화되지 않아 평문으로 전송되어 악의적인 사용자가 중간에서 데이터를 도청할 수 있는 스니핑 취약점이다.

## Low-Level

![그림 1-1](/assets/image/bwapp/sensitive%20data%20exposure/Clear%20Text%20HTTP%20(Credentials)/image.png)
- 일반적인 로그인 기능이 구현되어 있다.

![그림 1-2](/assets/image/bwapp/sensitive%20data%20exposure/Clear%20Text%20HTTP%20(Credentials)/image-1.png)
- 희생자 PC에서 bWAPP 서버의 로그인 페이로 이동
- 그 후 계정정보를 입력하여 로그인 하였다.

![그림 1-3](/assets/image/bwapp/sensitive%20data%20exposure/Clear%20Text%20HTTP%20(Credentials)/image-2.png)
- 그 후 동일 네트워크에서 기다리던 공격자 PC의 와이어샤크 패킷을 확인해보면
- 희생자 PC에서 입력한 값들이 그대로 평문으로 전송되어 노출되고 있다.

## High-Level

![그림 1-4](/assets/image/bwapp/sensitive%20data%20exposure/Clear%20Text%20HTTP%20(Credentials)/image-3.png)
- High level에서는 ssl/tls 통신을 하여 전송 데이터가
- 암호화되어 볼 수 없게된다.