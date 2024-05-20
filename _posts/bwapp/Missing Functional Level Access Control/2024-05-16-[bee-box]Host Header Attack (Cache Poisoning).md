---
layout: single
title: (bWAPP)Host Header Attack (Cache Poisoning)
categories: bwapp
tag: [Host Header Attack, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> 해당 취약점은 웹 서버가 HTTP요청 헤더인 host 헤더를 전적으로 신뢰하고 있을 경우 발생하며, 일반적으로 웹 서버는 요청이 도착한 호스트를 확인하기 위해 host헤더를 사용하지만 이를 조작할 경우 악의적인 사이트로의 이동을 우도할 수 있다.

![그림 1-1](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Host%20Header%20Attack%20(Cacche%20Poisoning)/image.png)
- 화면에 나오는 here 를 누르게되면 이전 페이지인 index.html로 이동하게 된다.

```
GET /bWAPP/portal.php HTTP/1.1
Host: 192.168.146.133
```

![그림 1-2](/assets/image/bwapp/Missing%20Functional%20Level%20Acess%20Control/Host%20Header%20Attack%20(Cache%20Poisoning)/image-1.png)
- 위 코드와 사진을 보게되면 Host 주소에 있는 ip를 기반으로 bWAPP/portal.php 파일로 이동하는 걸 알 수 있다.

![그림 1-3](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Host%20Header%20Attack%20(Cache%20Poisoning)/image-2.png)
- 이 때 페이지에서 Host 헤더의 값을 공격자 pc로 바꾸게 되면 a태그에 적용된 ip가 변경된 걸 볼 수 있으며, 페이지 또한, CSS파일을 제대로 참조할 수 없어 깨짐 현상이 발생하는 걸 볼 수 있다.

![그림 1-4](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Host%20Header%20Attack%20(Cache%20Poisoning)/image-3.png)
- 변조된 사이트에서 Host 헤더는 여전히 공격자 ip로 설정되어 있다.
- 이 때 here를 누르게되면 설정되어있는 공격자 ip를 기반으로 bWAPP/portal.php로 연결을 시도하게 된다.

![그림 1-5](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Host%20Header%20Attack%20(Cache%20Poisoning)/image-4.png)
- 사진과 같이 공격자 서버의 파일에 접근하여 php 파일을 다운받게 된다.
- 스푸핑, 파밍, 세션 하이재킹, csrf 등의 취약점으로 연계될 수 있다.