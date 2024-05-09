---
layout: single
title: (bWAPP)Denial-of-Service (XML Bomb)
categories: bwapp
tag: [DoS, xss, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명

> XML Bomb DoS는 XML을 이용하여 서버의 리소스를 과도하게 소모시켜 서비스를 마비시키는 DoS 공격의 일종이다. 이는 서버의 CPU 및 MEMORY 부하를 발생시킨다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(XML%20Bomb)/image.png)
- 아파치 웹 서버가 동작 중이며, xml dos에 취약하다고 한다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(XML%20Bomb)/image-1.png)
- 현재 bWAPP 서버의 CPU 및 MEMORY 상태이다. 낮은 사용량으로 확인된다.
- 제공되는 LOL 버튼을 클릭하고 와이어샤크 패킷을 확인해보면

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(XML%20Bomb)/image-2.png)
- 상위 엔터티를 지속적으로 호출하며 대량의 트래픽을 발생시키는 걸 확인할 수 있다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(XML%20Bomb)/image-3.png)
- 또한 bWAPP 서버의 CPU와 MEMORY 사용량을 보게되면 순식간에 모든 CPU와 MEMORY를 사용한 것을 볼 수 있다. 이로 인해 서버의 가용성이 저하된다.