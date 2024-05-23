---
layout: single
title: (bWAPP)CSRF (Change Password)
categories: bwapp
tag: [CSRF, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> CSRF 공격은 사용자의 의도와는 무관하게 공격자가 특정 웹 애플리케이션에 요청을 보낼 수 있게 하는 공격이며, 공격자가 사용자의 권한을 이용해 임의의 요청을 서버에 보낼 수 있습니다. 이로 인해 사용자가 의도하지 않은 작업이 수행될 수 있다.

![그림 1-1](/assets/image/bwapp/csrf/CSRF%20(Change%20Password)/image.png)
- 비밀번호 변경 로직이 존재한다.
- CSRF 공격의 경우 피해자가 세션이 유지된 상태에서 이루어져야한다. 즉, 피해자가 해당 웹 사이트에 로그인이 되어 있어야 의도치 않은 공격이 가능하다. 

```
/bWAPP/csrf_1.php?password_new=bug&password_conf=bug&action=change
```

- 해당 패스워드 변경시 GET 형식으로 URL을 통해 변경 로직이 전송되는 걸 볼 수 있다.
- 이 때 이전 패스워드 검사, CAPTCHA, CSRF토큰, Referer 헤더 검증등 어떠한 방어 기법이 적용되어 있지 않다.
- 즉, 해당 변경 로직을 희생자에게 전송하고 희생자가 클릭하게되면, 공격자가 의도한 패스워드로 강제 변경되게 된다.
- xss 취약점이 함께 존재할 경우, src속성을 이요한 악성 스크립트 제작 후 희생자에게 전송할 수 있다.