---
layout: single
title: (bWAPP)Reflected (Referer)
categories: bwapp
tag: [xss,reflected,stored,reflected xss, bee box,stored xss, OWASP TOP 10, OWASP, bwapp, dom xss]
toc: true
author_profile: false
---

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(Referer)-archive/image.png)
- 페이지 내에 Referer 헤더의 값이 고스란히 출력되고 있다.
- 버프스위트와 같은 도구를 사용하여 Request 값을 가로채
- 헤더 값을 변조하게 되면 스크립트가 발생한다.

```
Referer: <script>alert(1)</script>
```

![그림 1-1](/assets/image/bwapp/xss/Reflected%20(Referer)-archive/image-1.png)
- 요청 헤더를 변조함으로써 스크립트가 실행되었다.ㅊ