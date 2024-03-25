---
layout: single
title: (bWAPP)Session Mgmt. - Session ID in URL
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low (Mideum & Haigh 시나리오 없음)
![그림 1-1](/assets/image/bwapp/Broken-Auth/Session%20ID%20in%20URL-archive/image.png)
- 해당 시나리오에서는 사용자의 세션 값이 URI에 그대로 노출되어 발생할 수 있는 취약점을 다룬 시나리오이다.

![그림 1-2](/assets/image/bwapp/Broken-Auth/Session%20ID%20in%20URL-archive/image-1.png)
- 그림 1-2와 같이 URI 의 PHPSESSID 파라미터를 동해 GET 형식으로 세션값이 전달 되고 있다.
- 물론 현재 상용 사이트에서는 일어날 수 없는 일이지만, 위와 같은 취약점이 있을경우 매우 취약하다.

![그림 1-3](/assets/image/bwapp/Broken-Auth/Session%20ID%20in%20URL-archive/image-2.png)
- 실습은 이전 세션 취약점들과 동일하게, 서로 다른 브라우저를 통해
- bee 유저의 세션값을 탈취
- test 유저 페이지에서 세션값 변조를 통한Bee 유저로 로그인이 가능하다

> 세션 값은 절대 노출되지 않아야 하며, Http only, secure, expires, maxAge  등을 사용하여, 세션 관리가 이루어져야 한다.