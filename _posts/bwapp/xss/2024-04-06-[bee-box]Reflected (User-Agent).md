---
layout: single
title: (bWAPP)Reflected (User-Agent)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(User-Agent)-archive/image.png)
- 이전의 Referer 문제와 동일하게 User-Agent 헤더의 값이
- 페이지 내에 고스란히 출력되고 있다.
- 이 또한 Request 값을 가로채 요청 헤더의 값을 변조하면 스크립트가 실행된다.

```
User-Agent: <script>alert(1)</script>
```

![그림 1-2](/assets/image/bwapp/xss/Reflected%20(User-Agent)-archive/image-1.png)
- User-Agent 라는 요청 헤더를 변조함으로써 스크립트가 실행 된다.