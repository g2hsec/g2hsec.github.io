---
layout: single
title: (bWAPP)Base64 Encoding (Secret)
categories: bwapp
tag: [base64,sensitive data exposure, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> Base64는 암호화 방식이 아닌 인코딩, 디코딩 방식으로, 단 순 데이터 인코딩이다. 즉, 자체적인 보안 기능이 없으며, 누구나 쉽게 디코딩이 가능하다. 이 경우, 개인 정보 혹은 민감 정보를 Base64를 통해 인코딩 할 시 보안 위협으로 다가올 수 있다 중요 정보의 경우 인코딩이 아닌 암호화를 통해 관리해야 한다.

## Low-Level

![그림 1-1](/assets/image/bwapp/sensitive%20data%20exposure/Base64%20Encoding%20(Secret)/image.png)
- secret 정보가 암호화되어 쿠키에 저장되어 있다고 한다.

```
Cookie: security_level=0; PHPSESSID=858ff99b67d6f8bf2129419bf66ba15a; secret=QW55IGJ1Z3M%2F
```

- 해당 페이지의 쿠키값을 보기데묀 secret 속성에 특정 문자열이 보이게 된다.

![그림 1-2](/assets/image/bwapp/sensitive%20data%20exposure/Base64%20Encoding%20(Secret)/image-1.png)
- 해당 값을 디코딩 하게되면 secret 문자열이 고스란히 출력된다.
- 즉 해당 시나리오에서는 암호화를 한 것이 아닌 단순 Base64를 통한 인코딩을 통하여 seret값을 관리하고 있다.

## High-Level

```
Cookie: PHPSESSID=858ff99b67d6f8bf2129419bf66ba15a; security_level=2; secret=83785efbbef1b4cdf3260c5e6505f7d2261f738d
```
- high level에서는 base64가 아닌 다른 방식으로 관리 하고 있다.

![그림 1-3](/assets/image/bwapp/sensitive%20data%20exposure/Base64%20Encoding%20(Secret)/image-2.png)
- 확인 해보면 sha1 해시함수를 통환 해시화를 통해 관리하고 있는 듯 하다.
- md5 와 sha1 와 같은 해시 알고리즘은 오래되어 충분히 크랙당할 수 있는 취약한 해시 알고리즘이다.

![그림 1-4](/assets/image/bwapp/sensitive%20data%20exposure/Base64%20Encoding%20(Secret)/image-3.png)
- sha1 또한 크랙이 가능한 걸 확인할 수 있다.
- 즉 sha256과 같은 안전한 해시알고리즘을 사용해야 한다.