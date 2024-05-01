---
layout: single
title: (bWAPP)Insecure DOR - Insecure DOR (Reset Secret)
categories: bwapp
tag: [Insecure DOR, Access Contorol, IDOR, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low
![그림 1-1](/assets/image/bwapp/idor/reset-secret-archive/image.png)
- 사용자의 secret 값을 기본 default secret 값으로 변경하는 기능이 존재한다.

```
<reset><login>bee</login><secret>Any bugs?</secret></reset>
```

- Any bugs? 버튼과 함 께 secret 값이 변경 되는 요청 구문이 날라갈 때 
- request 값을 잡아보면 위와 같이 login 태그 요소가 bee 유저인 것을 볼 수 있다.

![그림 1-2](/assets/image/bwapp/idor/reset-secret-archive/image-1.png)
- 현재 test 사용자의 secret 값은 Test로 설정되어 있다.
- 이 때 IDOR 취약점을 이용하여 loing 태그의 요소를 bee가 아닌 test 사용자로 변경하요 요청을 보내 보았다.

![그림 1-3](/assets/image/bwapp/idor/reset-secret-archive/image-2.png)
- test 사용자의  secret 값이 Any bugs? 라는 default secret 값으로 변경된 걸 볼 수 있다.

