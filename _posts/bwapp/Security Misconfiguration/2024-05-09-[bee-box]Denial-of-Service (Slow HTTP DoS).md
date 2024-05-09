---
layout: single
title: (bWAPP)Denial-of-Service (Slow HTTP DoS)
categories: bwapp
tag: [DoS, xss, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> HTTP 요청에서 Header와 Body부분을 구분하기 위해 header의 마지막 부분을 표시하여야 한다. 이 때 사용되는 것이 CRLF인데 이러한 CRLF를 2번 보내어 메시지의 Header의 마지막을 선언해야하지만, CRLF문자를 1번만 보내 empty line를 포함시키지 않아 서버에게 더 보낼 데이터가 있는거 처럼 만들어 연결 상태를 유지시켜 가용성을 저하시킨다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(Slow%20HTTP%20DoS)/image.png)
- 아파치 웹 서버가 구동중이며. Slow HTTP DoS attacks에 취약한 것으로 보인다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(Slow%20HTTP%20DoS)/image-1.png)
- 일반적으로 정상적인 패킷을 보게되면 Content-Type: 와 같이 헤더의 마지막이 선언되면 \r\n\r\n(CRLF2번 실행)을 통해 Header와 Body를 구분하게 된다.

[사용 스크립트](https://github.com/gkbrk/slowloris)
<br>
Slow HTTP DoS 공격에 사용되는 스크립트를 사용하였다.

> bwapp 기준으로 스크립트 사용시 도메인 명을 사용 하여야 하므로, /etc/hosts파일에서 IP와 도메인을 매칭 해주어야 한다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(Slow%20HTTP%20DoS)/image-2.png)
- 지속적으로 커넥션을 맺기위한 작업이 이루어지고 있는 듯 하다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(Slow%20HTTP%20DoS)/image-3.png)
- 요청 패킷을 잡아 헤더를 보게되면 CRLF(0d0a) 가 한번만 사용된 걸 확인할 수 있다.

![그림 1-5](/assets/image/bwapp/Security%20Misconfiguration/Denial-of-Service%20(Slow%20HTTP%20DoS)/image-4.png)
- 또한 bWAPP 서버에서 네트워크 상태를 확인해보면 공격자 pc에서 80(http) 로의 tcp 커넥션 정보들이 매우 많이 존재한다. 이로 인해 서버는 통신과정에서 받을 데이터가 더 있는 것으로 여겨 지속적인 연결상태를 유지하므로 가용성이 저하된다.