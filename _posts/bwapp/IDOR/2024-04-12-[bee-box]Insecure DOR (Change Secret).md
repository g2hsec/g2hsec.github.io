---
layout: single
title: (bWAPP)Insecure DOR - Insecure DOR (Change Secret)
categories: bwapp
tag: [Insecure DOR, Access Contorol, IDOR, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/idor/change-secret-archive/image.png)
- 이전 xss 문제와 동일하게 특정 계정에 대한 secret값을 변경하는 부분이 존재한다.
- 해당 문제는 IDOR 취약점을 악용하는 문제이다.
- 수평적 권한 상승 및 변경을 통해 타인의 secret 값을 변경하는 듯 하다.

![그림 1-2](/assets/image/bwapp/idor/change-secret-archive/image-1.png)
- test 계정으로 로그인 하게되면
- 출력되는 secret 값이 Qwer 로 지정되어 있는 것을 알 수 있다.

```
secret=q&login=bee&action=change
```

- 다시 돌아와서 secret Change 부분에서 버프스위트를 통해 Request 값을 잡아보면
- secret 파라미터와 login 파라미터에 계정명이 들어가는 것을 확인할 수 있다.
- 이 때 login 값을 조작함으로써 타 사용자에 대한 secret 값 변조가 가능하다.

```
secret=q&login=test&action=change
```

- 위와 같이 파라미터 값을 변조 후 전송해보았다.

![그림 1-3](/assets/image/bwapp/idor/change-secret-archive/image-2.png)
- 우선 secret 값이 변경되었다는 문자열이 출력된다.

![그림 1-4](/assets/image/bwapp/idor/change-secret-archive/image-3.png)
- test 계정으로 로그인 해보면
- secret 값이 변조되어 q 로 출력되는 걸 확인할 수 있다.