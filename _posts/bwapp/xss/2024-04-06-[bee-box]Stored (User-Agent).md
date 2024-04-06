---
layout: single
title: (bWAPP)Stored (User-Agent)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---


![그림 1-1](/assets/image/bwapp/xss/Stored%20(User-Agent)-archive/image.png)
- 시간 날짜, 사용자의 ip, User-Agent 헤더의 냉용이 출력되는 걸 볼 수 있다.
- 이전 문제들과 동일하게 요청 헤더를 변조하면 스크립트가 실행되며
- 사용자의 접근 시간, ip, User-Agent 등의 값들으 서버측 데이터베이스에 저장 후 불러올 때 스크립트가 실행되게 된다.


```
User-Agent: test
```

![그림 1-2](/assets/image/bwapp/xss/Stored%20(User-Agent)-archive/image-1.png)
- 요청 헤더를 변조 후 전송시 변조된 헤더값이 출력되는 걸 볼 수 있다.

```
User-Agent:<script>alert(1)</script>
```

![그림 1-3](/assets/image/bwapp/xss/Stored%20(User-Agent)-archive/image-3.png)
![그림 1-4](/assets/image/bwapp/xss/Stored%20(User-Agent)-archive/image-2.png)
- 성공적으로 헤더값을 변조하여 스크립트가 실행되는 걸 알 수 있다.