---
layout: single
title: (bWAPP)Denial-of-Service (SSL-Exhaustion)
categories: bwapp
tag: [DoS, xss, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명

> DoS공격중 하나인 SSL-Exhaustion은  SSL/TLS 프로토콜을 통한 서버의 리소스를 고갈시켜 서비스를 중단시키는 공격이다. 이러한 공격은 서버가 SSL/TLS 핸드셰이크를 처리하는 동안 새로운 연결을 처리할 수 없어 서비스를 중단시키는 점을 악용한다. 즉, SSL 핸드셰이크 과정에서 클라이언트가 서버쪽으로 키 재요청을 플러딩 하게된다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(SSL-Exhaustion)/image.png)
- 포트 8443으로 동작중인 웹 서버는 SSL-Exhaustion 공격에 취약하다고 한다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(SSL-Exhaustion)/image-1.png)
- 와이어샤크를 통해 패킷을 확인해보면 SSL/TLS 통신을 하는것이 확인이 가능하다.

SSL-Exhaustion 공격을 수행하기 위해 thc-ssl-dos 라는 Tool을 사용하였다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(SSL-Exhaustion)/image-2.png)
![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(SSL-Exhaustion)/image-3.png)
- 해당 공격을 수행 후 패킷을 보게되면, 공격자 PC에서 지속적인 Client hello와 clinet key exchange와 같이 지속적인 플러딩을 수행하는 걸 확인할 수 있다.