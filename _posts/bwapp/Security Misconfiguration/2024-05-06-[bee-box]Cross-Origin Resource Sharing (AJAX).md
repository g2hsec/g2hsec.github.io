---
layout: single
title: (bWAPP)Cross-Origin Resource Sharing (AJAX)
categories: bwapp
tag: [cors,samba, sop, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> 해당 시나리오에서는 SOP 정책을 완화하기 위해 사용되는 CORS 적용이 잘 되어있는지 확인하는 시나리오이다.
> SOP란 서로 다른 출처 즉 다른 도메인에서의 공격을 방지하기 위한 웹 브라우저 보안 메커니즘이며, 다른 도메인의 경우 접근을 허용하지 않는다. 이 때 다양한 불편한 점을 완화 하기 위해 CORS를 사용하며 CORS는 HTTP 헤더에 기반하여 서로다른 Origin 간에 리소스를 공유하는 방법이다. 이 때 사용되는 헤더는 Access-Control-Allow-Origin 라는 응답헤더를 사용하여 해당되는 도메인으로부터의 리소스만 허용한다. 이 때 Access-Control-Allow-Origin 설정을 잘못 할 경우 취약한 상황이 발생한다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Cross-Origin%20Resource%20Sharing/image.png)
- ajax 요청을 사용하여 Neo의 비밀번호를 훔친다는 시나리오인 거 같다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Cross-Origin%20Resource%20Sharing/image-1.png)
- 아무런 문제 없이 secret 파일에 접근이 가능하다.

```
HTTP/1.1 200 OK
Date: Mon, 06 May 2024 00:37:36 GMT
Server: Apache/2.2.8 (Ubuntu) DAV/2 mod_fastcgi/2.4.6 PHP/5.2.4-2ubuntu5 with Suhosin-Patch mod_ssl/2.2.8 OpenSSL/0.9.8g
X-Powered-By: PHP/5.2.4-2ubuntu5
Access-Control-Allow-Origin: *
Content-Length: 51
Connection: close
Content-Type: text/plain

Neo's secret: Oh why didn't I took that BLACK pill?
```

- 해당 요청의 응답값을 보게 되면 Access-Control-Allow-origin: * 설정이 되어 있다. 이는 모든 도메인으로부터 리소스를 허용한다는 의미로 취약한 설정이다.

## Medium Level 및 High Level

```
if(isset($_SERVER["HTTP_ORIGIN"]) and $_SERVER["HTTP_ORIGIN"] == "http://intranet.itsecgames.com")
{

    header("Access-Control-Allow-Origin: http://intranet.itsecgames.com");

	echo "Wolverine's secret: What's a Magneto?";
	
}
```

- medium Level 에서는 Access-Control-Allow-Origin 헤더의 값으로 http://intranet.itsecgames.com 이 설정 되어 있다. 즉,  http://intranet.itsecgames.com 에서 들어오는 리소스만 허용한다는 의미로 적절히 보호하고 있다.

- High Level에서는 ACAO 헤더가 적용되어 있지 않다.